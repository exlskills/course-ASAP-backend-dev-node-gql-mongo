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

Note that if you already have `user_data_input` variable in your GraphiQL, add a comma after its JSON block and remove the closing and opening curly bracket in between the `user_data_input` and `order_data_input` - to set up your Query Variables section as a single JSON with two objects in it.

The mutation code in GraphiQL will be as follows:

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

As there are no validation on the User Order content in the demo, this mutation can be run multiple times, and new identical Documents will be added.

## Mutation Schema

The `src/relay-mutations/order/create-order-mutation.js` contains Input Object definition. 

The `mutateAndGetPayload` code reorganizes the input to convert Global IDs into those used in the database:

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

Notice, how `orderObj` is declared as a `const` and initialized with 2 parameters: `order_date` and an empty `order_items` array. Then 2 more parameters are added to it using the *dot* notation: `orderObj.user_id` and `orderObj.payer_id`. The Items array is populated in the *for* loop over the corresponding input data.


## Mutate-and-Get Function and The Database Processor

"Mutate and Get" function `createOrder` is in `src/relay-mutate-and-get/order-mag.js`:

- converts the `item_details` strings into JS objects via `JSON.parse()`. The conversion is placed into `try/catch` as this would generally be a place where problems with the inbound data format may break the application. In the demo code, the process exits on the first error. A common production-quality system would scan all records and report errors for the entire set at the end, if any
- calls `OrderCud.createOrder`
 
Database function `createOrder` is in `src/db-handlers/order/order-cud.js`:

- generates Order ID via `id_gen()`, validates that the ID has not been used already - by querying the database. Those devs used to transaction processing would wonder how can we guarantee that the ID would not be added by another parallel Order flow in the moments between we check for it and use it for creating our Document? We can't. Back to the discussion on what `id_gen()` does, though, the likelihood of this happening is statistically *zero*
- calls `create` method on the `UserOrder` model


Literally, that's all to do in there. We are done. Congratulations!