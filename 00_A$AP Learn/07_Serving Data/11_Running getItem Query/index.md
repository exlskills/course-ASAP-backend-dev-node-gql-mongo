### getItem Query

Let's review how a non-paging query is defined and works in GraphQL. Three things are different from the paging query design:

- no `...connectionArgs` in the query arguments
- the Object Type in the query is a `graphql` `GraphQLObjectType` vs. `graphql-relay` `connectionDefinitions`
- the query resolver calls the database query function directly vs. via `connectionFromDataSource`

The GraphQL query definition for `getItem` is in `src/relay-queries/item/get-item-query.js`, the Object Types are in `src/relay-models/item-type.js` and `src/relay-models/item-price-type.js` (per the database Schema, Item Price is an array embedded into the Item), the resolver `resolveGetItem` is in `src/relay-resolvers/item-resolver.js` and the database query processor `fetchById` is in `src/db-handlers/item-fetch.js`

The GraphiQL client side query can be found in `doc/graphql-grahiql-samples.md`:

```json
query getItem {
  getItem(item_id: "SXRlbToxR1ZsMjFscTBTOHA=") {
    id
    desc
    item_category
    item_price {
      amount
      discount_schema
      valid_from
      valid_to
    }
  }
}
```

## Item GraphQL Object Type Generic Structure

Notice that `src/relay-models/item-type.js` still contains the `connectionDefinitions` even though it is not used in the query. The purpose is very clear: the same Object Type can be utilized in `getItem` or `listItems` (not coded in the demo), or even as a sub-element in something like the `listUserOrdersDetails` where the all Item fields are displayed. Just like everything else in programming, Object Types should be kept generic and reused when possible

## Item ID Filter Definition 
Per the demo `src/relay-queries/item/get-item-query.js`, the `getItem` query receives one required argument - Item Global ID:

```
    item_id: {
      type: new GraphQLNonNull(GraphQLID)
    }
```

## Display and Filter of ItemPrice

`src/relay-models/item-type.js` defines `item_price` as a `GraphQLList`. In the real business use case, We'll likely want to provide capabilities query for a specific price, e.g., *active* vs. *historical*, or *price on date*. This can be done in multiple ways (not coded in the demo):

- if we keep the design to a single database query, we would need to add arguments to `src/relay-queries/item/get-item-query.js` controlling the price extract logic and pass into the database query. The return would only contain applicable price record(s), however, the logic will run regardless whether the `item_price` field was requested by the client or not
- in the spirit of GraphQL, complex evaluations related to a specific field in the output should be placed in the *field* resolver. If we do so for the `item_price` logic, we would likely need to code a separate database query for that, which may not be affecting the performance too much as the Item Document would be retrieved by its `_id` using the default Collection index. Keep in mind that the GraphQL query arguments are not propagated into *field* resolvers, only the `obj` is, so in this design we either have to provide separate arguments for the `item_price` evaluation (making it a sub-query, as shown in `src/relay-models/user-order-type.js` for `order_items`) or pass the necessary evaluation conditions by plugging them in into the array element of the object returned by the *query* resolver, which does have access to the `args`
- another spin on this can be taken by utilizing the `info` object we mentioned before - the fourth and generally unused argument of the query resolver. Take a look at the output of this debug logging added to `src/relay-resolvers/item-resolver.js`: 
  
  ```
  logger.debug(` info.fieldNodes ` + JSON.stringify(info.fieldNodes));
  if (
    info.fieldNodes &&
    info.fieldNodes.length > 0 &&
    info.fieldNodes[0].selectionSet
  ) {
    for (let fld of info.fieldNodes[0].selectionSet.selections) {
      logger.debug(` field ` + JSON.stringify(fld));
    }
  }
  ```

  It shows how the list of fields requested by the client can be extracted from the `info` object. When we run `getItem` from GraphiQL, the following will be seen in the logging output:

  ```
  <timestamp> debug:  field {"kind":"Field","name":{"kind":"Name","value":"id","loc":{"start":775,"end":777}},"arguments":[],"directives":[],"loc":{"start":775,"end":777}}
  <timestamp> debug:  field {"kind":"Field","name":{"kind":"Name","value":"desc","loc":{"start":782,"end":786}},"arguments":[],"directives":[],"loc":{"start":782,"end":786}}
  <timestamp> debug:  field {"kind":"Field","name":{"kind":"Name","value":"item_category","loc":{"start":791,"end":804}},"arguments":[],"directives":[],"loc":{"start":791,"end":804}}
  <timestamp> debug:  field {"kind":"Field","name":{"kind":"Name","value":"item_price","loc":{"start":809,"end":819}},"arguments":[],"directives":[],"selectionSet":{"kind":"SelectionSet","selections":[{"kind":"Field","name":{"kind":"Name","value":"amount","loc":{"start":828,"end":834}},"arguments":[],"directives":[],"loc":{"start":828,"end":834}},{"kind":"Field","name":{"kind":"Name","value":"discount_schema","loc":{"start":841,"end":856}},"arguments":[],"directives":[],"loc":{"start":841,"end":856}},{"kind":"Field","name":{"kind":"Name","value":"valid_from","loc":{"start":863,"end":873}},"arguments":[],"directives":[],"loc":{"start":863,"end":873}},{"kind":"Field","name":{"kind":"Name","value":"valid_to","loc":{"start":880,"end":888}},"arguments":[],"directives":[],"loc":{"start":880,"end":888}}],"loc":{"start":820,"end":894}},"loc":{"start":809,"end":894}}
  ```

  `info.fieldNodes[0].selectionSet.selections[i].name.value` is what we can check to see if `item_price` was requested by the client - and bypass its evaluation in the main database query if it was not

- last point to mention: it probably makes more sense setting `item_price` in the Item Object Type as a *connection* vs. a *list*, enabling the GraphQL paging functionality. As a rule, datasets that may grow in size should have paging enabled. As we saw, *paging* is optional to use on the client side even if it is enabled in the server schema design, and it adds minimal overhead in general processing use cases


Before moving on to Mutations, let's check out a "free form" query and also take a pick at how `mongoose` *object* can potentially interfere with our JS code flow

