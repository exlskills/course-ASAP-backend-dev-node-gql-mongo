### GraphQL Mutation Schema

Check out `src/relay-mutations/order/create-order-mutation.js`

There are a few objects defined in the file that are used closely together. The actual *mutation* definition part is here :

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

Note that class `mutationWithClientMutationId` is imported from `graphql-relay` package, so it is Relay-specific.

Although the syntax looks different, this structure has the same general components as a query definition: input, processing function, output.

In `mutateAndGetPayload`, there are three arguments being passed into the processing function: `(inputFields, viewer, info)`. This is similar to a query `resolve` property in queries, where four arguments are passed `(obj, args, viewer, info)`. As the the concept of *source* `obj` does not apply to mutations, `obj` is absent in `mutateAndGetPayload`. 

Inside the function in `mutateAndGetPayload`, Relay Global IDs are converted into the MongoDB IDs, then `src/relay-mutate-and-get/order-mag.js` `createOrder` is called. The reason the ID conversion is done here vs. in `createOrder` is to isolate GraphQL-specific work from the business and DB logic, whenever possible.

Mutation input is usually wrapped as *Input Types*. `OrderInputType` is defined in the same JS file, above the mutation definition. Somewhat similar to Object Types that map resolver output to the data sent to the client, Input Types act as an extra mapping layer to perform necessary conversions on the incoming client data before it is passed into `mutateAndGetPayload`. 

`outputFields` are similar to the Object Type in queries - they are loaded from the object returned by `mutateAndGetPayload`. In this case, we assuring that `createOrder` returns an object matching the structure defined in `outputFields`.


Enough with the overview, let's move on to setting up and running the demo project's code next!
