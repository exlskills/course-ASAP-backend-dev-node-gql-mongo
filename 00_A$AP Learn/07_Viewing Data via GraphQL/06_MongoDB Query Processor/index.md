### MongoDB Query Processor - The End Of The Line

Congratulations, this is the last topic to learn about building a GraphQL server on NodeJS with a MongoDB backend! At least on the Query side, but the Update (mutation) side is very similar, not much new in there after this.

Following the path from the demo `listUsers` GraphQL Query schema in `src/relay-queries/user/user-list-query.js`, which defines the resolver, through `src/relay-resolvers/user-resolver.js`, which defines the database query processor as `queryFunction: fetchUserOrderList`, we get to `src/db-handlers/user/user-order-list-fetch.js`, where we find

```
export const fetchUserOrderList = async (
  filterValues,
  aggregateArray,
  viewerLocale,
  fetchParameters
) => {
```

The list of parameters is driven by the logic coded in the pagination processor `src/paging-processor/find-with-paging.js`. For non-paging queries, we can call the database query processing function directly from the the query resolver and pass other parameters as required. To recap the "Paging Flow JS Code" lesson, in this demo design the following parameters have been included into the call from the pagination wrapper:

- `filterValues` is reserved for the free-form query string that we may be passing from the client (we don't in the demo)
- `viewerLocale` is an example of some important general `viewer` information we may frequently need to use when querying the database
- `fetchParameters` are assembled in the query resolver, logically, from the query arguments
- and, finally, `aggregateArray` is the `sort`/`skip`/`limit` component we apply to the query based on the client request and pagination logic 

Inside `fetchUserOrderList` function, the logic is disappointingly simple. We use the straight-forward `mongoose` `find` method on the `User` model, with no conditions or *projections* to control which fields are included into the extract. Then we add the sort/skip/limit methods:

```
  let queryFunc = User.find();

  const sort = aggregateArray.find(item => !!item.$sort);
  if (sort) {
    queryFunc = queryFunc.sort(sort.$sort);
  }
  const skip = aggregateArray.find(item => !!item.$skip);
  if (skip) {
    queryFunc = queryFunc.skip(skip.$skip);
  }
  const limit = aggregateArray.find(item => !!item.$limit);
  if (limit) {
    queryFunc = queryFunc.limit(limit.$limit);
  }
```

As an example, for the 4th page of users, sorted by `full_name`, the `queryFunc` may look like this:
```
User.find().sort({full_name: 1}).skip(30).limit(10)
```
The actual query evaluated from `queryFunc` and sent by `mongoose` to the MongoDB engine will be listed in the `Mongoose:` debug logging output:
```
Mongoose: user.find({}, { sort: { full_name: 1, username: 1 }, skip: 30, limit: 10, projection: {} })
```

The query execution line must be put into its own `try/catch` block:
```
  try {
    userArray = await queryFunc.exec();
  } catch (err) {
    logger.error(`in fetchUserList exec: ` + err);
    throw new Error(`query failed`)
  }
```

## Error Handling 

Although, `try/catch` behavior in JS is similar to that in Java, where you can put a large chunk of code in the `try` part and expect getting the `catch` part executed if any line inside the `try` fails, in NodeJS it looks like the engine sometimes chokes up if an `await` line in the middle of a long `try` section throws and error or *rejects the Promise*. Rather than reaching the catch, the engine may complain of an *unhandled promise rejection*. Particularly, this may be happening if a `mongoose` `await` fails. Considering that `await` is more a syntactical way to *pause* the JS flow and wait for the result than the actual implementation of the async processes in the engine, these side effects need to be taken in consideration and addressed so we can use the code reliably. Wrapping each `mongoose` `await` in its own `try/catch` seems to be doing the trick, so it is generally used throughout the demo code.

In the example above, we *log* the actual error so we can act on it in the log monitor, but don't return error details to the client. This is a generally recommended practice protecting server internals from exposure to outside systems. As the GraphQL runs in the debug mode itself, we'll get a detailed error stack leading to the `throw` line, visible in GraphiQL, but in production the client will only receive the `query failed` message. The message can obviously be expanded to include some recommended steps, e.g., the support email. In a multi-language system, the `throw` statement would use a code pointing to a message translation map.

A more controlled approach can be adding a custom completion code field to all GraphQL queries and set it's value accordingly to indicate if an error was encountered. In such a design, all errors in the flow should be *caught* and an empty data reply with the error code set should be returned instead of throwing an error to the query resolver. The demo uses this approach when processing Migrations. For Queries, the deign relies on the built-in error handling of the GraphQL engine that wraps errors into `errors` object of the response.

The behavior can be tested by overriding the `return userArray;` line at the end of `fetchUserOrderList` with something like `return new Error('test error')`

## The DB Query Response

In `mongoose`, the response from the query can be an array of objects or a single object, as well as an empty array or `null` if the query found no matching records. Arguably, for consistency of coding, it would be reasonable to expect a uniform reply in all cases, but, let's not bet on this to happen - think of how much code out there has been produced relying on the existing output format. As a simple rule, `findOne` and `findById` return *one* record by design, or `null` if nothing was found, whereas `find` or *aggregations* return an array of objects or an empty array.

The debug logging statement at the end of prints this 
```
 userArray 
  { primary_locale: 'en', _id: '1E4Yo11Y3r9a', full_name: 'Jane Doe', username: 'jane01',
  primary_email: 'jd@example.com', created_at: <date-time>Z, updated_at: <date-time>Z, __v: 0 },
  { primary_locale: 'en', _id: '1Eg6d1aFL8s7', full_name: 'John Public', username: 'john01',
  primary_email: 'jp@example.com', created_at: <date-time>Z, updated_at: <date-time>Z, __v: 0 }
```

`__v: 0` is a special Document versioning field in `mongoose`. The demo doesn't use it, but it is present in the schemas by default and gets included into the database query output when we don't explicitly control the list of fields via the *projection*. 

Following the reverse logic flow from the database query response, via the query resolver to the Object Type, the GraphQL engine will load, for each element in the array, the matching GraphQL response fields with the data and convert `_id` into GraphQL Global ID following this instruction in `src/relay-models/user-type.js`:
```
id: globalIdField('User', obj => obj._id),
```

The `edge`, `node` and `cursor` get populated by the logic coded in `src/paging-processor/connection-from-datasource.js`.

Lastly, `interfaces: [NodeInterface]` in the `src/relay-models/user-type.js` links to some code in `src/relay-models/node-definition-types.js`, which is not maintained in the demo. By the GraphQL engine design, this interface can be used to provide a default method of retrieving a GraphQL model from the MongoDB database. This functionality is not used in the demo - all queries have resolvers that implement the fetching. However, the definition is included into the Object Type schema.

`listUsers` looks very simple to implement - and it is. Let's now look at a "complex" query that lists Orders and Order Items for a User. If you recall from previous chapters, Order Items will act as a *sub-query*, with its *own paging*. Sounds very difficult. Well, you'll be disappointed again - it is very simple, border line primitive, to code. With JS and MongoDB - what else to expect?
