### getItem Query

Let's review a non-paging GraphQL query. These things are different from paging queries we worked with so far:

- no `...connectionArgs` in the query arguments
- the Object Type in the query is a `graphql` `GraphQLObjectType` vs. `graphql-relay` `connectionDefinitions`
- the query resolver calls the database query function directly vs. via `connectionFromDataSource`
- the query resolver output should be an object vs. an array, unless the Object Type is a List

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

Notice that `src/relay-models/item-type.js` still contains the `connectionDefinitions` even though it is not used in the query. The purpose is very clear: the same Object Type can be utilized in `getItem` or `listItems` (not coded in the demo project), or even as a sub-element in something like the `listUserOrdersDetails` where all Item fields are displayed. Object Types should be kept generic and reused when possible


## Item ID Filter Definition 

Per `src/relay-queries/item/get-item-query.js`, the `getItem` query receives one required argument - Item Global ID:

```
    item_id: {
      type: new GraphQLNonNull(GraphQLID)
    }
```

## Display and Filter of the Item Price

As defined in the Item database schema `src/db-models/item-model.js`, the Item Price is an embedded array of ItemPrice objects defined in `src/db-models/item-price-model.js`. Similarly, `src/relay-models/item-type.js` defines `item_price` as a `GraphQLList`. 

The database query in `src/db-handlers/item-fetch.js` simply pulls the `item` Collection record with the `item_id`, which includes all prices entered for item.

In a real business use case, we'd likely want to provide capabilities to retrieve a specific price, e.g., *active* vs. *historical*, or *price on date*. This can be done in multiple ways (not coded in the demo project):

- keep the single database query design, passing arguments to drive the price extract logic. The downside is that the logic will run regardless whether the `item_price` field was requested by the client or not
- in the spirit of GraphQL, place field-specific evaluation logic into the *field* resolver. Define `item_price` as a sub-query, similarly to `order_items` in `src/relay-models/user-order-type.js` and code a separate database query to extract the price from the embedded array
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

  The `for` loop above demonstrates how the list of fields requested by the client can be extracted from the `info` object. 

  Here's the logging output:

  ```
  <timestamp> debug:  field {"kind":"Field","name":{"kind":"Name","value":"id","loc":{"start":775,"end":777}},"arguments":[],"directives":[],"loc":{"start":775,"end":777}}
  <timestamp> debug:  field {"kind":"Field","name":{"kind":"Name","value":"desc","loc":{"start":782,"end":786}},"arguments":[],"directives":[],"loc":{"start":782,"end":786}}
  <timestamp> debug:  field {"kind":"Field","name":{"kind":"Name","value":"item_category","loc":{"start":791,"end":804}},"arguments":[],"directives":[],"loc":{"start":791,"end":804}}
  <timestamp> debug:  field {"kind":"Field","name":{"kind":"Name","value":"item_price","loc":{"start":809,"end":819}},"arguments":[],"directives":[],"selectionSet":{"kind":"SelectionSet","selections":[{"kind":"Field","name":{"kind":"Name","value":"amount","loc":{"start":828,"end":834}},"arguments":[],"directives":[],"loc":{"start":828,"end":834}},{"kind":"Field","name":{"kind":"Name","value":"discount_schema","loc":{"start":841,"end":856}},"arguments":[],"directives":[],"loc":{"start":841,"end":856}},{"kind":"Field","name":{"kind":"Name","value":"valid_from","loc":{"start":863,"end":873}},"arguments":[],"directives":[],"loc":{"start":863,"end":873}},{"kind":"Field","name":{"kind":"Name","value":"valid_to","loc":{"start":880,"end":888}},"arguments":[],"directives":[],"loc":{"start":880,"end":888}}],"loc":{"start":820,"end":894}},"loc":{"start":809,"end":894}}
  ```

  `info.fieldNodes[0].selectionSet.selections[i].name.value` is what we would check to see if the `item_price` was requested by the client. If it wasn't - bypass the price evaluation logic.

- last point to mention: it probably makes sense setting `item_price` in the Item Object Type as a *connection* vs. a *list*, enabling the GraphQL paging functionality. As a rule, datasets that may grow in size should have paging enabled. As we saw, *paging* is optional to use on the client side even if it is enabled in the server's schema design.


Finally, before moving on to Mutations, let's check out the "client-defined" query option. We'll also dig a bit deeper into the structure of the `mongoose` query result *object* 

