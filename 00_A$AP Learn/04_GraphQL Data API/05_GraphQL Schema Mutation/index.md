### Sample GraphQL Mutation Schema

See `src/relay-mutations/order/create-order-mutation.js`

The *mutation* definition part is here (note, here we use a relay-specific class `mutationWithClientMutationId` imported from `graphql-relay` package):
```
export default mutationWithClientMutationId({
  name: 'CreateOrder',
  inputFields: {
    order_data: {
      type: OrderInputType
    }
  },
  outputFields: {
    order_id: { type: GraphQLID },
    completionObj: { type: CompletionObjType }
  },
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
});
```

Although the syntax looks different, the structure has the same components as a query definition: input, output, processing function.

`mutateAndGetPayload` here is coded slightly differently from `resolve` in queries. In `resolve`, we were calling an async function passing the four parameters `(obj, args, viewer, info)`. Here, we have three parameters going into the processing function - `(inputFields, viewer, info)` - as the concept of *source* `obj` does not apply. 

The middle part of `mutateAndGetPayload` converts Relay Global IDs into the MongoDB IDs. The reason it's done here vs. in `createOrder` is to isolate GraphQL-specific work (like `fromGlobalId`) from the business and DB logic, whenever possible.

Mutation input is usually wrapped as *Input Types* - here, we use `OrderInputType`, defined in the same JS file, above the mutation definition. Similarly to Object Types, Input Types act as a buffer and help controlling any necessary conversions before the input data is passed into `mutateAndGetPayload`. 

`outputFields` are similar to the Object Type in queries - they are loaded from the object returned by `mutateAndGetPayload`. In this case, we assuring that `createOrder` returns an object matching the structure defined in `outputFields`.

Enough with the "overview", lets dive into the code now!
