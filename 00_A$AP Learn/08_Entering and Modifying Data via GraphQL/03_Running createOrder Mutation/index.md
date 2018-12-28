### createOrder Mutation

Load the `QUERY VARIABLES` section of your GraphiQL browser window with this content from `doc/graphql-grahiql-samples.md`, Mutations, Create Order, Variables:

```json
{
  "order_data_input": {
    "order_data": {
      "user_id": "VXNlcjoxRTRZbzExWTNyOWE=",
      "payer_id": "VXNlcjoxRTRZbzExWTNyOWE=",
      "order_date": "2018-09-10T00:00:00.000Z",
      "order_items": [
        {
          "item_id": "SXRlbToxSEVPeDZGbkM3Y00=",
          "quantity": 5,
          "amount": 10.13,
          "item_details": "{\"packaging\": \"frozen-pack\", \"special_handling\": {\"before_shipping\": \"freeze\", \"in_transit\":\"refrigerate\"}}"
        },
        {
          "item_id": "SXRlbToxR1ZsMjFscTBTOHA=",
          "quantity": 2,
          "amount": 15.99,
          "item_details": "{\"color\": \"black\"}"
        }
      ]
    }
  }
}
```

Note that if you already have `user_data_input` variable in your GraphiQL, add a comma after its JSON block and remove the closing and opening curly bracket in between the `user_data_input` and `order_data_input` - to have your Query Variables section look as a single JSON object with two variables in it.

The mutation code in GraphiQL is as follows (copy it from `doc/graphql-grahiql-samples.md` as well):

```
mutation sendOrder($order_data_input: CreateOrderInput!) {
  createOrder(input: $order_data_input) {
    order_id
    completionObj {
      code
      msg_id
      msg
      processed
      modified
    }
  }
}
```

In the demo project, no duplicate validation is coded on the User Order content, so this mutation can be run multiple times with the same data - new identical Documents will be created.


## Mutation Schema

The `src/relay-mutations/order/create-order-mutation.js` contains the Input Object definition. 

`mutateAndGetPayload` reorganizes the input and converts Global IDs into those used in the database:

```
  mutateAndGetPayload: (inputFields, viewer, info) => {
    logger.debug(`in mutateAndGetPayload Create Order `);
    logger.debug(`  inputFields ` + JSON.stringify(inputFields));
    const orderObj = {
      order_date: inputFields.order_data.order_date,
      order_items: []
    };
    orderObj.user_id = fromGlobalId(inputFields.order_data.user_id).id;
    orderObj.payer_id = fromGlobalId(inputFields.order_data.payer_id).id;
    for (let item of inputFields.order_data.order_items) {
      const itemObj = { ...item };
      itemObj.item_id = fromGlobalId(item.item_id).id;
      orderObj.order_items.push(itemObj);
    }
    return createOrder(orderObj, viewer);
  }
```

Notice, how `orderObj` is declared as a `const` and initialized with 2 parameters: `order_date` and an empty `order_items` array. Then two more parameters are added to it using the *dot* notation: `orderObj.user_id` and `orderObj.payer_id`. The Items array is populated in the `for` loop over the corresponding input data.


## Mutate-and-Get Function and The Database Processor

"Mutate and Get" function `createOrder` is in `src/relay-mutate-and-get/order-mag.js`:

- converts the `item_details` strings into JS objects via `JSON.parse()`. The conversion is placed into `try/catch` as this would generally be a place where problems with the inbound data format may break the application. In the demo code, the process exits on the first error. A common production-quality system would scan all records and report errors for the entire set at the end, if any
- calls `OrderCud.createOrder`
 
Database function `createOrder` is in `src/db-handlers/order/order-cud.js`. 


## Order ID Validation Loop and MongoDB Error Checking

Similarly to `createUser`, the Order ID is generated via `id_gen()`. However, in this flow we don't validate the uniqueness of the ID by querying the database. Instead, the `while` loop is coded wrapping the `create` method execution on the `UserOrder` model. If the ID is not unique, the `create` will error out with a message like this:

```
MongoError: E11000 duplicate key error collection: web_dev.user_order index: _id_ dup key: { : "1UtNnZBYUHDx" }
```

`mongoose` documentation recommends using this check when trapping unique index violation errors:

```
err.message.indexOf('duplicate key error') !== -1
```

Sounds great. If the documentation suggests this as the *official* approach, vs., e.g., somehow extracting the error code, etc. - we're using it. The only problem, if we have multiple unique indexes in the Collection we need to make sure the problem is with the `_id` field - that we can fix by re-generating - vs. with another field that we should report back to the client as a hard stop error. 

As MongoDB reports the troubled `key` back - we can compare it with the `_id`, so the final error trapping condition is 

```
  if (
        err.message.indexOf(order_id) !== -1 &&
        err.message.indexOf('duplicate key error') !== -1
      )
``` 


This completes the development tasks. The very last step is to generate the client-side GraphQL schema extract!