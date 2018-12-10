### What Is In The Query Resolver

The last part of a GraphQL query definition is the `resolver`. E.g., in the `src/relay-queries/user/user-list-query.js`, we have:
```
resolve: (obj, args, viewer, info) =>
    resolveListUsers(obj, args, viewer, info)
```

This is the actual *function* that executes the query on the GraphQL server - like the *Controller* in *MVC*. As you learned in the coding Elementary School, code should be modularized, so instead of writing JS commands right in the query definition (as we could have) - we put those in a separate imported file `src/relay-resolvers/user-resolver.js`. 

We've looked inside `resolveListUsers` already when reviewing the pagination implementation. In the demo project, *query resolvers* are simply an intermediate layer between the query structure and the database layer, and the idea is that if the database layer changes significantly, e.g., away from MongoDB to something else - the GraphQL side is not getting affected. Just a nice design principle.

What is important about the resolvers, is the parameters that the GraphQL engine passes into them: the four-member `obj, args, viewer, info` group. We detailed each of those already, so just to recoup:

- `obj` - a GraphQL engine generated object as the output from other query resolvers that might have already run in the overall query processing flow. Applies to queries that are executed inside outlying queries, otherwise - null. Let's wait till we review `listUserOrders` to see how this parameter is used
- `args` - this is what the client sent us as the *query arguments* 
- `viewer` - this is the GraphQL engine controlled *context* that we configured to be the object that identifies the *user* executing the query on the client side
- `info` - a raw detailed representation of the query, we don't use it as we have everything we need organized by the engine into other parameters already, but we are passing it just in case we would ever need it for something
 
## Viewer-based Security and Validation

The resolver is a good place to code a security layer that would control access to the query function by *viewer* as well as cleanse sensitive data as the DB query returns its result applying some viewer-specific logic. E.g., here we can control that a standard User can only see his/her orders, whereas a Shipping Manager can query all orders in his/her line of work. 

Note, that as individual fields each have their individual *resolver* running after the *query resolver* completion (or the *default* one if none specified in the field definition) - outbound field-level security can be coded in the field resolvers

## Arguments Validation

Generally, the schema should be set up with the base query arguments validation built in. E.g., we can use `GraphQLNonNull` to mark required arguments, control the argument type and *enumeration* content. More complex validation can be put into the query resolver

## GraphQLID to Internal ID Conversion

`base64` encoded GraphQL IDs can be converted to their corresponding internal IDs in the query resolver by executing `graphql-relay` function `fromGlobalId`. Remember, the conversion back to the GraphQL ID is best done in the field resolver


Next, passing through the resolver into the MongoDB query processor code!