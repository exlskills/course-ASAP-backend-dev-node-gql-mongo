### GraphQL Object Type Component

Object Types are key components of the GraphQL schema. They are used (referred to) in queries to define the structure of the output that the query produces. As mentioned previously, Object Types carry out a dual function: in addition to providing the list of fields, they also specify the evaluation (mapping) rules for each field in a form of a JS *resolver* function. The rules application step is run after the completion of the query processing function - the query *resolver*, and it is based on the query resolver's output object - referred to as the *source* for the Object Type. 

The GraphQL engine executes the *resolver* for each Object Type field individually. The *source* object is passed into the function as its parameter.

In practice, only some fields of the Object Type have resolvers coded - the majority would use the *default resolver*, which matches the name of the field with the content of the *source* object and maps the value from the source's matching field, if found (otherwise, the value would be sent as a `null`). So, when the field names are consistent across MongoDB and GraphQL Object Types - the matching works almost seamlessly. We'll see it in practice in the demo.

In the example `src/relay-models/order-item-type.js` imports come from `graphql`, `graphql-relay`, as well as `./node-definitions-type`. The last two are relay-specific, we'll discuss them in details later. As mentioned, the `relay` part covers paging through the dataset in a more advanced way than the core `graphql` package does.

Two components are exported - the Object Type and the *Connection* for paging and navigation over it's content:
```
export const OrderItemType = new GraphQLObjectType({
...
export const { connectionType: OrderItemConnection } = connectionDefinitions({
  name: 'OrderItem',
  nodeType: OrderItemType
});
```

Type's `interfaces: [NodeInterface]` property is another component related to the paging implementation.

The `fields` property is, technically, a JS function that returns an object containing definitions of all fields and optional *resolver* functions. 

`id` and `item_id` have resolvers - calls to `globalIdField`:
```
    id: globalIdField('OrderItem', obj => obj._id),
    item_id: globalIdField('Item', obj => obj.item_id),
```
`obj` in the above represents the *source*, which this design assumes to be the MongoDB-retrieved Order Item object - see the familiar MongoDB model field names.

Note on `globalIdField()` function: just like MongoDB requires unique IDs for its Documents, Relay uses [Base64 encoded](https://www.base64encode.org/) IDs internally. The IDs *must be* unique, otherwise, the Relay client will misbehave. Relay calls those Global IDs, as they also contain a prefix defining the Object, e.g., `'OrderItem'`, in addition to what needs to be a unique Order Item ID. `globalIdField()` is provided by `graphql-relay` package to facilitate the ID encoding; there is a corresponding function for decoding of Global IDs.

The rest of `fields` matches the MongoDB model, so that the *default resolver* can map the values in from the extracted *source*

## GraphQL Object Type and Connection Embedding
`src/relay-models/user-order-type.js` illustrates how embedded objects can be handled in GraphQL: 
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

Note `type: OrderItemConnection`, which refers to the above described `src/relay-models/order-item-type.js`.

The entire `order_items:` property has a lot of stuff in it, and it basically represents a query syntax that we will review next. As we said, in the schema different kinds of elements blend in together.

Let's think about where this complexity is coming from. In the MongoDB schema, Order Items is an array of objects within the User Order Document. There is an equivalent way to define arrays (lists) of embedded objects in GraphQL, which would look like this:
```
  order_items: {
    type: new GraphQLList(OrderItemType)
  }
```
Literally, the above defines `order_items` as a list (array) of `OrderItemType`. The `OrderItemConnection` is not used in this case.

As there is no limit on how many Order Items a User Order may have, there may be orders with lots of them. That may overflow a small mobile device screen and strain the client app memory. A more proper solution is to always treat any List as a *Connection* we can page through.

The `args` should be provided for the `order_items` paging specifically, similarly how those are provided when running a query - we'll look at the GraphQL query definition next
