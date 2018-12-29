### GraphQL Object Type Schema Component

GraphQL schema Object Types are used in query schemas to define the structure of the *output* that the query produces. 

As mentioned in the previous lesson, Object Types carry out a dual function: 

- provide the list of fields in an Object, e.g., User Object has `id`, `full_name`, `username`, etc.
- specify the evaluation (mapping) rules for each field in a form of a JS function
 
The evaluation step is executed by the GraphQL engine after the completion of the query processing function. Query-level functions are called *query resolver* - we'll see them defined in the `resolve` property of queries. Correspondingly, field-level functions defined in Object Types are often referred to as *field resolvers*.

In the scope of the given query processing, the return of the query resolver is passed as an argument into each field resolver. The query resolver output is referred to as the *source* for the Object Type processing. The GraphQL engine executes the *resolver* for each Object Type field individually. 

Field resolvers are unaware of each other and cannot communicate directly. Knowing from the JS overview chapter that objects passed as arguments can be modified inside functions, field resolvers may, in theory, change the content of the *source* as they run, but the sequence of the field resolver execution is not exactly documented. Interested should check the source code of the `graphql` package for opportunities. Others - assume that each field resolver acts on its own. Also, keep in mind that if the client hasn't requested a field to be included into the output - its resolver will not run. 

In practice, only some fields in Object Types have resolvers coded - the majority would use the *default resolver*, provided by the engine, which *matches* the name of the Object Type field with the content of the *source* object and maps the value from the source's matching field, if found, or sends out a `null` otherwise. So, here's a tip: when  field names are consistent across MongoDB Schemas and GraphQL Object Types - the matching from the mongoDB extract and the GraphQL output works automatically. We'll see it in practice in the demo.

Just a notch deeper into the theory, as everything in JS is a *function*, the *default resolver* doesn't just read a value from the *source* object when it finds a matching name - it *executes* the value as if it was a function, which it can be. So, here's another opportunity building some data-driven complex solutions - out of scope in the demo project.

### Example: Order Item Type

`src/relay-models/order-item-type.js`. The imports come from `graphql`, `graphql-relay`, as well as `./node-definitions-type`. The last two are relay-specific, we'll discuss them in details later. The `relay`-specific part of the demo project implementation covers the *paging* functionality in a more specialized way than the core `graphql` package defines.

Two components are exported - the Order Item Object Type and the *Connection* for paging and navigation over it's content:

```
export const OrderItemType = new GraphQLObjectType({
...
export const { connectionType: OrderItemConnection } = connectionDefinitions({
  name: 'OrderItem',
  nodeType: OrderItemType
});
```

Type's `interfaces: [NodeInterface]` property is another component related to the paging implementation.

The `fields` property is a JS function that returns an object containing definitions of all fields and optional field-level *resolver* functions. 

`id` and `item_id` have resolvers - calls to `globalIdField`:

```
    id: globalIdField('OrderItem', obj => obj._id),
    item_id: globalIdField('Item', obj => obj.item_id),
```

`obj` here represents the *source* returned from the query resolver. We'll review that function in later chapters, but you can see the familiar MongoDB schema field names `_id` and `item_id`. Yes, the output of the query resolver will be a MongoDB extract.

### GraphQL Relay Global ID

Just like MongoDB requires that Documents have unique IDs, GraphQL Relay uses its own internal IDs to identify all objects. In the above example, each Order Item is assumed to have an `id` and Item - `item_id`. In the Relay client, the data processing and caching logic is based on identifying data by those IDs. The IDs *must be* unique, otherwise, the Relay client will misbehave. Note that GraphQL does not enforce the uniqueness, so it is up to the developer to ensure that the IDs are generated properly in the dataset being sent to the client.

The IDs are [Base64 encoded](https://www.base64encode.org/). Relay calls them Global IDs: they consist of a prefix defining the Object, e.g., `'OrderItem'`, and a unique identifying the instance of the Object, e.g., Order Item ID. So, combined together, the prefix plus the ID are *globally* unique.

Function `globalIdField()` is provided by `graphql-relay` package to facilitate the ID encoding into Base64, as well as a corresponding function for decoding from Base64 back to the readable Global IDs form. Base64 is an HTTP-friendly format and widely used in web development.

### Other Fields

The rest of `fields` in `src/relay-models/order-item-type.js` matches the MongoDB schema, so that the *default resolver* can map the values in from the extracted *source*

### GraphQL Object Type and Connection Embedding - User Order Type

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

Here `type: OrderItemConnection` refers to the above described `src/relay-models/order-item-type.js`.

The entire `order_items:` property has a lot of stuff in it, and it basically represents a query syntax that we will review next. As we said, in the GraphQL schema, different kinds of elements blend in together.

Let's think about where this complexity is coming from. In the MongoDB schema, Order Items is an array of objects within the User Order Document. There is an equivalent way to define arrays (lists) of embedded objects in GraphQL, which would look like this:

```
  order_items: {
    type: new GraphQLList(OrderItemType)
  }
```

Literally, the above defines `order_items` as a list (array) of `OrderItemType`. The `OrderItemConnection` is not used in this case.

As there is no limit on how many Order Items a User Order may have, there may be orders with lots of them. That may overflow a small mobile device screen and strain the client app memory. A proper design is to always treat any List as a *Connection* that the client can retrieve page-by-page.
<br>
Next, we'll review Query definitions. You'll see what the `args` are, etc. 
