### Fetching User Order Items from MongoDB 

As outlined earlier, the User Order-level and Order Item-level data is extracted via separate query resolvers, which in turn, logically, use separate database query processors. 

## Retrieving Order-level fields via `find`

Although in MongoDB order and item data are combined in a single Document, *projection* enables us pulling selected fields of the document only. In `src/db-handlers/user/user-order-list-fetch.js`, the `find` statement has two additions compare to what we saw in `src/relay-queries/user/user-list-query.js`:

```
let queryFunc = UserOrder.find({ user_id: fetchParameters.user_id }).select({
    order_items: 0
  });
```

- filter `{ user_id: fetchParameters.user_id }` inside `find` itself
- and `select({order_items: 0})` after `find`

The filter value `fetchParameters.user_id` is passed from the GraphQL query argument `user_id` into the `fetchParameters` object in the resolver `src/relay-resolvers/user-order-resolver.js`, after convertion from the Global ID and pre-validating against the User Collection:

```
  const user_id_db = fromGlobalId(args.user_id).id;
  const user_rec = await fetchById(user_id_db, { _id: 1 });
  if (!user_rec || !user_rec._id) {
    return Promise.reject('User not found');
  }
  const fetchParameters = { user_id: user_id_db };
```

Note, that the pre-validation is done by calling an imported function vs. using `mongoose` data layer functionality directly in the resolver.

The `select` can be used to either list the fields to include into the `mongoose` query output (marking them with a `1`) or exclude (`0`). In this case, `order_items` are excluded


## Fetching Order Items via MongoDB Aggregation Query

`src/db-handlers/order/order-item-list-fetch.js` demonstrates how *aggregation* is used in MongoDB queries. You've been warned in the "MongoDB Datastore" chapter that Aggregation is as huge of a topic and complex as advanced SQL Query and something like PL SQL combined. Luckily, the *80-20* paradigm works well for backend development with MongoDB Aggregation: 80% of functionality can be covered with 20% of commands. Even more like *90-10*.

`fetchOrderItemList` well illustrates the approach, syntax and covers key methods used in MongoDB Aggregation:

- aggregation is *pipeline*: a step-by-step transformation of data, where each step is represented as an element of the aggregation array. In `fetchOrderItemList`, an empty array is initialized along with a variable used to build its elements: 

  ```
  let elem,
    agrArray = [];
  ```

  Additionally, `try/catch` blocks are placed around the array building steps and, separately, the `exec` call (as explained in "MongoDB Query Processor" lesson in this chapter)
- per MongoDB aggregation syntax, all command / key words inside statements are prefixed with `$`. Mongo schema field names on the *right side* of the assignment statements are also prefixed with the `$`.
- for testing and development, the pipeline can stopped at any point by commenting out the rest of it, up to but not including the `exec`. The logging statement after `exec` will print the intermediate output
- a pipeline usually starts with a `$match` command that acts as a filter to limit the number of Documents coming into the pipeline right from the beginning
- `$project` is used to control which fields are needed in the subsequent steps, however, the engine would pull in missing fields if required as well as remove those that don't affect the final output. So, in reality, intermediate `$project` statements, if any, are used for streamlining field names, consolidating multiple or adding calculated fields, and the final `$project` is the only one required to limit what comes out
- `$unwind` is necessary to *flatten* embedded arrays to work with their elements in subsequent steps at the individual record level. E.g., in `fetchOrderItemList` we `$unwind` the Order Items before we *join* the Item Collection to pull in details required in the final output
- `$lookup` is the equivalent of SQL *join*. In the latest versions of MongoDB `$lookup` is designed nicely to allow expansive filtering options on how the join works, by executing a *sub pipeline* on the external Collection

# Aggregation Steps Detailed Review

The best way to get familiar with how Aggregation works is to review live code and try it in action. Let's walk through the actual Order Item Aggregation pipeline flow from `fetchOrderItemList`. 

To remind ourselves what this query processor does - it extracts User Order Items for the given User Order, as defined in `src/relay-models/user-order-type.js`, `order_items` field. The Order ID is taken from the *source* `obj`, which is the output from the parent query, and passed via the sub-query resolver into `fetchOrderItemList`.

The aggregation is done on the `UserOrder` Model - the Model name is specified in the execution step at the end of the function: 
```
orderItemArray = await UserOrder.aggregate(agrArray).exec();
```

- Step 1. Select the UserOrder Document using the Order ID that was taken from the `obj` and loaded into `fetchParameters`:
  
  ```
  elem = {
      $match: {
        _id: fetchParameters._id
      }
    };
  agrArray.push(elem);
  ```

  Note, the `push` statement adds the `elem` to the `agrArray` - nothing else happens at this point. The call to the database is done once, at the very end, by launching the shown above `exec()` on the `aggregate` method of the `UserOrder` Model, passing the `agrArray`. So the steps and actions described reference what *will* happen when the query is passed by `mongoose` and run by the MongoDB engine

  The output of this step will be an array of one element - the complete User Order Document. Remember, aggregate queries in MongoDB always return *array* objects, regardless of how many records have been extracted:

  ```
  [{"_id":"20V4vVvGvdOA","user_id":"1E4Yo11Y3r9a","payer_id":"1E4Yo11Y3r9a","order_date":"<date-time>Z","order_items":[{"_id":"5c0f529bb1c54f55dbbdc050","item_id":"1HEOx6FnC7cM","quantity":5,"amount":10.13,"item_details":{"packaging":"frozen-pack","special_handling":{"before_shipping":"freeze","in_transit":"refrigerate"}},"created_at":"<date-time>Z","updated_at":"<date-time>Z"},{"_id":"5c0f529bb1c54f55dbbdc04f","item_id":"1GVl21lq0S8p","quantity":2,"amount":15.99,"item_details":{"color":"black"},"created_at":"<date-time>Z","updated_at":"<date-time>Z"}],"created_at":"<date-time>Z","updated_at":"<date-time>Z","__v":0}]
  ```

- Step 2. For further processing, leave only the `order_item` field in scope - the embedded array of Order Item objects. As mentioned, this is an optional step as the query processing engine controls which fields should be included into the pipeline based on the flow and its final structure

  ```
  elem = {
      $project: {
        _id: 0,
        order_items: 1
      }
    };
  array.push(elem);
  ```
  
  The output is much cleaner, though, than from Step 1:
  
  ```
  [{"order_items":[{"_id":"5bf7bf9f97f8e75c21bb6a36","item_id":"1HEOx6FnC7cM","quantity":5,"amount":10.13,"item_details":{"packaging":"frozen-pack","special_handling":{"before_shipping":"freeze","in_transit":"refrigerate"}},"created_at":"<date-time>Z","updated_at":"<date-time>Z"},{"_id":"5bf7bf9f97f8e75c21bb6a35","item_id":"1GVl21lq0S8p","quantity":2,"amount":15.99,"item_details":{"color":"black"},"created_at":"<date-time>Z","updated_at":"<date-time>Z"}]}]
  ```
  
  Notice, how `order_items` is presented as an embedded array

- Step 3. *Unwind* the `order_item` array 

  ```
  elem = {
      $unwind: '$order_items'
    };
  array.push(elem);
  ``` 

  No more embedded array, but the output array has two elements now. `order_items` field that used to point to the embedded array, is now present in each element of the output array and points to the objects that used to be inside the embedded array. This is what `$unwind` does - it creates multiple *records*, equal to the number of the array elements, attaching the content that was *outside* of the array to each individual element of the array. If we haven't removed the extra fields in Step 2, the output would look much longer and harder to read. Although, again, from the engine's stand point, the Step 2 didn't make a difference:

  ```
  [{"order_items":{"_id":"5bf7bf9f97f8e75c21bb6a36","item_id":"1HEOx6FnC7cM","quantity":5,"amount":10.13,"item_details":{"packaging":"frozen-pack","special_handling":{"before_shipping":"freeze","in_transit":"refrigerate"}},"created_at":"<date-time>Z","updated_at":"<date-time>Z"}},
   {"order_items":{"_id":"5bf7bf9f97f8e75c21bb6a35","item_id":"1GVl21lq0S8p","quantity":2,"amount":15.99,"item_details":{"color":"black"},"created_at":"<date-time>Z","updated_at":"<date-time>Z"}}]
  ```

- Step 4. Further clean up the intermediate output structure, by removing extra fields and dropping `order_items` from the remaining field names. Technically, the name change is done by creating new fields and assigning values from the `order_items` embedded fields to them

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

  Notice, we use `item_details_as_ordered` not to collide with `item_details` that we are going to pull from the Item Collection

- Step 5. Prepare the *join* (*lookup* in MongoDB terminology) logic for the Item Collection. Notice how we initialize and use another aggregate array here: `array_lookup`

  * 5a. Set the join condition - it will be a `$match` evaluated as an *expression* `$expr` that compares two values for their equality `$eq`. One value is the `_id` field of the Item Collection (the one being *looked up*) and the other is a variable (notice the `$$`) that we'll assign in the lookup statement later

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

    The output of this step is hard to capture as it is executed deep inside the engine, but it is pretty clear what it does - filters the Item by item_id from the Order Item line

    * 5b. Then set the fields we need in our master dataset when the other collection is joined in

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

- Step 6. Execute the `lookup`. The Collection is specified by its MongoDB name (`item`) vs. `mongoose` schema; the `let` parameters assigns the value to the `item_id` variable used in `array_lookup` with the `$$` prefix; the `pipeline` is set as `array_lookup`, and the prefix the incoming data will get is going to be `item`

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

  The `item` stuff got attached as an array (aggregations always return arrays), so we need to unwind it now

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

- Step 8. Set up the final output structure by removing the `item.` level

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

  This is the final output from the aggregation

  ```json
  [
  {"item_id":"1HEOx6FnC7cM","quantity":5,"amount":10.13,"item_details_as_ordered":{"packaging":"frozen-pack","special_handling":{"before_shipping":"freeze","in_transit":"refrigerate"}},"desc":"Sample Food Item","item_category":"Food","item_details_base":{"shelf_life_mo":3,"weight_lb":0.41,"packaging":"vacuum"}},
  {"item_id":"1GVl21lq0S8p","quantity":2,"amount":15.99,"item_details_as_ordered":{"color":"black"},"desc":"Sample Consumer Item","item_category":"Consumer","item_details_base":{"weight_lb":1.3,"color":"green"}}
  ]
  ```

- Step 9. Except, the sort/skip/limit should be applied to the end of the aggregation array as well

  ```
    const sort = aggregateArray.find(item => !!item.$sort);
    if (sort) array.push(sort);
    const skip = aggregateArray.find(item => !!item.$skip);
    if (skip) array.push(skip);
    const limit = aggregateArray.find(item => !!item.$limit);
    if (limit) array.push(limit);
  ```

Once the aggregation is executed, we need to merge `item_details_as_ordered` with `item_details_base`, overriding the *base* info from the Item Collection with what's requested on the Order Item, where the parameters match. JS does it automatically for us, and then we convert the `item_details` from an object to a JSON compliant string and drop the unneeded values from the output object:

```
    for (let orderItem of orderItemArray) {
      orderItem.item_details = JSON.stringify({
        ...orderItem.item_details_base,
        ...orderItem.item_details_as_ordered
    });
    delete orderItem.item_details_base;
    delete orderItem.item_details_as_ordered;
```

And the final output is:

```json
[{"item_id":"1HEOx6FnC7cM","quantity":5,"amount":10.13,"desc":"Sample Food Item","item_category":"Food","item_details":"{\"shelf_life_mo\":3,\"weight_lb\":0.41,\"packaging\":\"frozen-pack\",\"special_handling\":{\"before_shipping\":\"freeze\",\"in_transit\":\"refrigerate\"}}"},{"item_id":"1GVl21lq0S8p","quantity":2,"amount":15.99,"desc":"Sample Consumer Item","item_category":"Consumer","item_details":"{\"weight_lb\":1.3,\"color\":\"black\"}"}]
```

The logic is very clear. The syntax - requires a lot of using to after SQL. The rules on what should be a variable, an expression, or just a simple object in each statement are confusing - if the dev doesn't know what they are. But once your first aggregation has been put together and working - the following are largely copy-and-paste


Before we move to Mutations, let's take a look at a simple GraphQL query with no paging, to relax after the aggregations dump