### Aggregation Steps Detailed Review

Let's walk through the Order Item Aggregation pipeline flow coded in `src/db-handlers/order/order-item-list-fetch.js` `fetchOrderItemList`. But, first, recap the use case and content of the surrounding code components.

The database query extracts User Order Items for the given User Order, per the GraphQL schema defined in the `order_items` field of UserOrderType Object Type in `src/relay-models/user-order-type.js`. 

`order_items` is mapped to a GraphQL *sub-query*, which is resolved via `resolveListOrderItems` and feeds `OrderItemConnection` Object Type. The sub-query is processed by the *paging* engine. The arguments to the sub-query must be passed by the GraphQL client explicitly as they are separate from those passed into the parent's query.  

In `resolveListOrderItems`, the Order ID for the database query is taken from the *source* `obj`, which is the output of the parent query, and passed into `fetchOrderItemList` via `fetchParameters`.

### Aggregation Pipeline

The aggregation in `fetchOrderItemList` is done on the `UserOrder` Model - see the Model name specified in the execution step at the end of the function's code: 

```
orderItemArray = await UserOrder.aggregate(agrArray).exec();
```

The aggregation pipeline steps are as follows:

- Step 1. Select the UserOrder Document using the Order ID loaded into `fetchParameters`:
  
  ```
  elem = {
      $match: {
        _id: fetchParameters._id
      }
    };
  agrArray.push(elem);
  ```

  Note, the `push(elem)` statement adds the `elem` into the `agrArray`, but no execution happens at this point. The call to the database is done once, after the array is fully formed, by launching the shown above `exec()` statement. So, the write-up describes what *will* happen when the query is passed by `mongoose` into the database and executed by the MongoDB engine.

  The output of the Step 1 will be an array of one element - the complete User Order Document. Remember, aggregate queries in MongoDB always return *array* objects, regardless of how many records have been extracted:

  ```
  [{"_id":"20V4vVvGvdOA","user_id":"1E4Yo11Y3r9a","payer_id":"1E4Yo11Y3r9a","order_date":"<date-time>Z","order_items":[{"_id":"5c0f529bb1c54f55dbbdc050","item_id":"1HEOx6FnC7cM","quantity":5,"amount":10.13,"item_details":{"packaging":"frozen-pack","special_handling":{"before_shipping":"freeze","in_transit":"refrigerate"}},"created_at":"<date-time>Z","updated_at":"<date-time>Z"},{"_id":"5c0f529bb1c54f55dbbdc04f","item_id":"1GVl21lq0S8p","quantity":2,"amount":15.99,"item_details":{"color":"black"},"created_at":"<date-time>Z","updated_at":"<date-time>Z"}],"created_at":"<date-time>Z","updated_at":"<date-time>Z","__v":0}]
  ```

- Step 2. For further processing, leave only the `order_item` field - the embedded array of Order Item objects. As mentioned, this is an optional step as the query processing engine itself controls which fields should be included into the pipeline based on the logic and the final output structure

  ```
  elem = {
      $project: {
        _id: 0,
        order_items: 1
      }
    };
  array.push(elem);
  ```
  
  The output is much cleaner, though, than from Step 1 - nice for testing:
  
  ```
  [{"order_items":[{"_id":"5bf7bf9f97f8e75c21bb6a36","item_id":"1HEOx6FnC7cM","quantity":5,"amount":10.13,"item_details":{"packaging":"frozen-pack","special_handling":{"before_shipping":"freeze","in_transit":"refrigerate"}},"created_at":"<date-time>Z","updated_at":"<date-time>Z"},{"_id":"5bf7bf9f97f8e75c21bb6a35","item_id":"1GVl21lq0S8p","quantity":2,"amount":15.99,"item_details":{"color":"black"},"created_at":"<date-time>Z","updated_at":"<date-time>Z"}]}]
  ```
  
  Notice `order_items` is an *embedded* array in the above output

- Step 3. *Unwind* the `order_item` array 

  ```
  elem = {
      $unwind: '$order_items'
    };
  array.push(elem);
  ``` 

  ```
  [{"order_items":{"_id":"5bf7bf9f97f8e75c21bb6a36","item_id":"1HEOx6FnC7cM","quantity":5,"amount":10.13,"item_details":{"packaging":"frozen-pack","special_handling":{"before_shipping":"freeze","in_transit":"refrigerate"}},"created_at":"<date-time>Z","updated_at":"<date-time>Z"}},
   {"order_items":{"_id":"5bf7bf9f97f8e75c21bb6a35","item_id":"1GVl21lq0S8p","quantity":2,"amount":15.99,"item_details":{"color":"black"},"created_at":"<date-time>Z","updated_at":"<date-time>Z"}}]
  ```
  
  No more embedded array, but the output itself became a two elements array. The `order_items` field that used to point to the embedded array, is now present in each element of the output array and points to the objects that used to be inside the embedded array. This is what `$unwind` does - it creates multiple *records*, equal to the number of the array elements, attaching the content that was *outside* of the array to each individual element of the array. If we haven't removed the extra fields in Step 2, the output would look much longer and harder to read. Although, again, from the engine's stand point, the Step 2 didn't make a difference

- Step 4. Further clean up the intermediate output structure by removing extra fields and dropping `order_items` *prefix* from the field names. Technically, the name change is done by creating new fields and assigning values from the `order_items` elements to them

  ```
  elem = {
      $project: {
        item_id: '$order_items.item_id',
        quantity: '$order_items.quantity',
        amount: '$order_items.amount',
        item_details_as_ordered: '$order_items.item_details'
      }
    };
  array.push(elem);
  ```
  
  Looks much cleaner now:

  ```
  [{"item_id":"1HEOx6FnC7cM","quantity":5,"amount":10.13,"item_details_as_ordered":{"packaging":"frozen-pack","special_handling":{"before_shipping":"freeze","in_transit":"refrigerate"}}},{"item_id":"1GVl21lq0S8p","quantity":2,"amount":15.99,"item_details_as_ordered":{"color":"black"}}]
  ```

  Notice, we named the field `item_details_as_ordered` not to collide with the `item_details` that we are going to pull from the Item Collection

- Step 5. Prepare the *join* (*lookup* in MongoDB terminology) logic for the Item Collection. Notice how we initialize and use another aggregate array `array_lookup` here. The lookup pipeline is completely separate, it is executed within the main pipeline as a specific step, and specific data from the main pipeline is passed into the lookup one as *variables* to be used in steps coded into the lookup pipeline. Somewhat convoluted at first, but makes sense: rather than coding a *lookup* condition to some separate syntax, the *join* is executed as another pipeline, using the same general syntax as in the main one and utilizing the same powerful commands. Just pass the variables

  * 5a. Set the join condition - it will be a `$match` evaluated as an *expression* `$expr` that compares two values for their equality `$eq`. One value is the `_id` field of the Item Collection (the one being *looked up*) and the other is the variable (notice the `$$`) `$$item_id` that we'll assign in the actual statement that launches the lookup pipeline

    ```
    let array_lookup = [];
    elem = {
      $match: {
        $expr: {
          $eq: ['$_id', '$$item_id']
        }
      }
    };
    array_lookup.push(elem);
    ```

    The output of this step is hard to capture as it is executed deep inside the engine, but it is pretty clear what it does - filters the Item by item_id that comes from the corresponding Order Item line array element

    * 5b. Then set the fields we need to pull in from the *looked-up* Collection into the main dataset

    ```
    elem = {
      $project: {
        desc: 1,
        item_category: 1,
        item_details: 1
      }
    };
    array_lookup.push(elem);
    ```

- Step 6. Execute the `lookup`. The Collection is specified by its MongoDB name (`item`) vs. `mongoose` schema; the `let` parameters assigns the value to the `item_id` variable used in `array_lookup` with the `$$` prefix; the `pipeline` is set as `array_lookup`, and the prefix the pulled-in data will get in the main dataset is going to be `item`

  ```
  elem = {
      $lookup: {
        from: 'item',
        let: { item_id: '$item_id' },
        pipeline: array_lookup,
        as: 'item'
      }
    };
  array.push(elem);
  ```

  Here's the result:

  ```json
  [{"item_id":"1HEOx6FnC7cM","quantity":5,"amount":10.13,"item_details_as_ordered":{"packaging":"frozen-pack","special_handling":{"before_shipping":"freeze","in_transit":"refrigerate"}},"item":[{"_id":"1HEOx6FnC7cM","desc":"Sample Food Item","item_category":"Food","item_details":{"shelf_life_mo":3,"weight_lb":0.41,"packaging":"vacuum"}}]},{"item_id":"1GVl21lq0S8p","quantity":2,"amount":15.99,"item_details_as_ordered":{"color":"black"},"item":[{"_id":"1GVl21lq0S8p","desc":"Sample Consumer Item","item_category":"Consumer","item_details":{"weight_lb":1.3,"color":"green"}}]}]
  ```

  See, the `item` stuff got pulled in into each element as an array (remember, pipeline aggregations always return arrays), so we need to unwind it now

- Step 7. Unwind the Item info

  ```
  elem = {
      $unwind: '$item'
    };
  array.push(elem);
  ```

  The number of records (array elements) in the output didn't change, but without unwinding the structure would be more difficult to work with

  ```json
  [
  {"item_id":"1HEOx6FnC7cM","quantity":5,"amount":10.13,"item_details_as_ordered":{"packaging":"frozen-pack","special_handling":{"before_shipping":"freeze","in_transit":"refrigerate"}},"item":{"_id":"1HEOx6FnC7cM","desc":"Sample Food Item","item_category":"Food","item_details":{"shelf_life_mo":3,"weight_lb":0.41,"packaging":"vacuum"}}},
  {"item_id":"1GVl21lq0S8p","quantity":2,"amount":15.99,"item_details_as_ordered":{"color":"black"},"item":{"_id":"1GVl21lq0S8p","desc":"Sample Consumer Item","item_category":"Consumer","item_details":{"weight_lb":1.3,"color":"green"}}}
  ]
  ```

- Step 8. Set up the final output structure by removing the `item.` prefix

  ```
  elem = {
      $project: {
        item_id: 1,
        quantity: 1,
        amount: 1,
        item_details_as_ordered: 1,
        desc: '$item.desc',
        item_category: '$item.item_category',
        item_details_base: '$item.item_details'
      }
    };
    array.push(elem);
  ```

  This is (almost) the final output from the aggregation

  ```json
  [
  {"item_id":"1HEOx6FnC7cM","quantity":5,"amount":10.13,"item_details_as_ordered":{"packaging":"frozen-pack","special_handling":{"before_shipping":"freeze","in_transit":"refrigerate"}},"desc":"Sample Food Item","item_category":"Food","item_details_base":{"shelf_life_mo":3,"weight_lb":0.41,"packaging":"vacuum"}},
  {"item_id":"1GVl21lq0S8p","quantity":2,"amount":15.99,"item_details_as_ordered":{"color":"black"},"desc":"Sample Consumer Item","item_category":"Consumer","item_details_base":{"weight_lb":1.3,"color":"green"}}
  ]
  ```

- Step 9. Lastly, any sort/skip/limit should be applied to the end of the aggregation array

  ```
    const sort = aggregateArray.find(item => !!item.$sort);
    if (sort) array.push(sort);
    const skip = aggregateArray.find(item => !!item.$skip);
    if (skip) array.push(skip);
    const limit = aggregateArray.find(item => !!item.$limit);
    if (limit) array.push(limit);
  ```

### Ta-da

```
orderItemArray = await UserOrder.aggregate(agrArray).exec();
```

### Post-processing - Merging Item Details

Once the aggregation is executed, we need to merge `item_details_as_ordered` with `item_details_base`. For the matching parameters, we need to override the *base* info from the Item Collection with what was requested on the Order Item. JS does it automatically for us when the subsequent *spread* `...` of the two structures is used. Then we convert the `item_details` from an object to a JSON compliant string via `JSON.stringify` and drop the unneeded values from the output object:

```
    for (let orderItem of orderItemArray) {
      orderItem.item_details = JSON.stringify({
        ...orderItem.item_details_base,
        ...orderItem.item_details_as_ordered
    });
    delete orderItem.item_details_base;
    delete orderItem.item_details_as_ordered;
```

The final output is:

```json
[{"item_id":"1HEOx6FnC7cM","quantity":5,"amount":10.13,"desc":"Sample Food Item","item_category":"Food","item_details":"{\"shelf_life_mo\":3,\"weight_lb\":0.41,\"packaging\":\"frozen-pack\",\"special_handling\":{\"before_shipping\":\"freeze\",\"in_transit\":\"refrigerate\"}}"},{"item_id":"1GVl21lq0S8p","quantity":2,"amount":15.99,"desc":"Sample Consumer Item","item_category":"Consumer","item_details":"{\"weight_lb\":1.3,\"color\":\"black\"}"}]
```

The logic of the aggregation is straightforward and clear. The syntax requires a lot of using-to after SQL. The rules on what should be a variable, an expression, or just a simple object in each statement are confusing at first. But once your first aggregation has been put together and working - the following are largely copy-and-paste.
<br>
Let's take a look at a simple GraphQL query with no paging - to relax a bit after the aggregations brain dump