### listUserOrders Code Components and Process Flow

## Acknowledging the Graph Theory

As outlined before, the line between Query and Object Type schemas in GraphQL is pretty blurred: both have *resolvers* and may receive *source* objects from previous steps in the flow. Why is that? Lack of design separation?

Those who know the meaning of term `graph` in Computer Science, may realize that *GraphQL* should have something to do with it. Adding `edges` and `nodes` - we have [Graph Theory](https://en.wikipedia.org/wiki/Graph_theory) written all over it. What does it mean for the JS dev? GraphQL is good for *drilling in* and unearthing data linked to the data we already have, literally. In a relational structure, we *join* tables when we need to retrieve a set of linked together data. Say, in the User Order use case, we'd join the 2 tables - Order and OrderItem - in a SELECT statement and produce a *page* of a REST extract with all information in it. We would not even think of doing it in any way differently.

In GraphQL, we don't *join* pieces of information - rather, we follow a graph-like structure to find *downstream* pieces linked to those that we already have. In the end, we get the same data out, don't be fooled about that. It's just about *how* we get the data and what advantages one way introduces vs. the other.

So, if we are going after User Order with Order Items, first, we get the Order and then we get the Order Items that comprise the order: two steps vs. one *join* step. If you don't think there is a big difference - you are probably right. It's just the way the code and the flow is structured. A key advantage, though, of doing it in two steps - depending on the user query, we may not need to run the second step at all, or run it partially only. This fits very well into a browsing use case when we want to show some parts of a multi-layered dataset to the user and offer an opportunity to ask for more from a particular layer (think "paging")


## listUserOrders GraphQL Query and Object Types

- query: `src/relay-queries/user/user-order-list-query.js`
  * takes the same arguments we've seen before: `orderBy`, `filterValues`, `resolverArgs`, `...connectionArgs`, plus the ID of the user in the form of GlobalId, a required argument:

  ```
   user_id: {
      type: new GraphQLNonNull(GraphQLID)
    }
  ```

  * returns data per `UserOrderConnection` Object Type
  * gets processed by `resolveListUserOrders(obj, args, viewer, info)`
- object types:
  * `src/relay-models/user-order-type.js` defines `UserOrderConnection` over `UserOrderType` that contains the Order-level fields we have in the demo, *plus* a field that represents a *sub-query* to get the Order Items:

  ```
  order_items: {
      type: OrderItemConnection,
      args: {
        orderBy: {
          type: inputTypes.OrderByType
        },
        filterValues: {
          type: inputTypes.FilterValuesType
        },
        resolverArgs: {
          type: inputTypes.QueryResolverArgsType
        },
        ...connectionArgs
      },
      resolve: (obj, args, viewer, info) =>
        resolveListOrderItems(obj, args, viewer, info),
      description: 'Order items'
    }
  ```
  Here we have the familiar set of arguments we pass to a paging query and another query resolver call, this time to `resolveListOrderItems(obj, args, viewer, info)`
  
  Note, though, that we don't include the Order ID into the sub-query arguments. How does `resolveListOrderItems` know which Order Items to retrieve? This is where the *source* `obj` comes into play. Think about it: according to the GraphQL flow, the *parent's* resolver `resolveListUserOrders` completes first - it produces an array of orders as its output object. To get from the query resolver output to the GraphQL response, each record from the array gets processed through the Object Type logic where it acts as the *source* to Object Type's *field level* resolvers. 

  One of the those field level resolvers - `order_items` - happens to be a query on its own. The `obj` becomes an argument of that query, and that is where the Order ID comes from. 
  
  Not much different than how *source* `obj` is used in one-liner resolvers like this one for the `user_id` field:
  ```
  user_id: globalIdField('User', obj => obj.user_id)
  ```
  
  Other than the `obj`, though, nothing else gets passed from the parent query. All the sub-query arguments separate from those for the parent query, even though the have the same names.


Remember, we learned that GraphQL will only return data that the client asked for, otherwise, the field-level resolvers are not engaged. Here's a great example to prove that: if the client does not ask for `order_items` - `resolveListOrderItems` will not be called. What a difference from coding REST with table joins! Sure, we could have provide two separate REST end points: one for Order-level and another for Order Item-level data queries. But imagine the amount of work on both server and client to separate those. Here, one query covers all use cases, and if the client asks for *more* OrderItems via this query - the client-side caching *may* realize that it already has the Order-level data and only the sub-query will run on the server. Or may be not - we can't really test it within the scope of the demo as we're not developing a true Rest client here. Anyway, there is enough stuff to work with as-is.


Let's run `listUserOrders` now from our GraphiQL browser tool!