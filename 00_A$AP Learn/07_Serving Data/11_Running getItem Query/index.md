### getItem Query

Let's review how a non-paging query is defined and works in GraphQL. Three things are different from the paging query design:

- no `...connectionArgs` in the query arguments
- the Object Type in the query is a `graphql` `GraphQLObjectType` vs. `graphql-relay` `connectionDefinitions`
- the query resolver calls the database query function directly vs. via `connectionFromDataSource`

The GraphQL query definition for `getItem` is in `src/relay-queries/item/get-item-query.js`, the Object Types are in `src/relay-models/item-type.js` and `src/relay-models/item-price-type.js` (per the database Schema, Item Price is an array embedded into the Item), the resolver is in `src/relay-resolvers/item-resolver.js` and the database query processor in `src/db-handlers/item-fetch.js`

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

- if we keep the design to a single database query, we would need to add arguments to `src/relay-queries/item/get-item-query.js` controlling the price extract logic and pass into the database query. The return would only contain applicable price record(s)

 