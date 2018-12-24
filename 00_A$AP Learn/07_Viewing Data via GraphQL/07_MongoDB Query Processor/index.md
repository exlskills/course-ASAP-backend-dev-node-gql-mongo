### MongoDB Query Processor - The Bottom Of The Flow Curve

Congratulations, this is the last *new* topic to learn about building a GraphQL server on NodeJS with a MongoDB backend! As mutations are quite similar to queries, you won't see much new after this.

Writing MongoDB queries is generally a somewhat specialized skill. There used to be a SQL Developer skillset - a person who knows SQL real well and at least one vendor-specific tool like PL/SQL. Not that MongoDB is so much easier that SQL. It's just that if your MongoDB queries become as complex as multi-page PL/SQL *procedures* - there is likely something fundamentally wrong with the platform selection and app design for your use case. You can do pretty complex things in MongoDB queries (as we briefly learned in the "MongoDB Datastore") chapter. You don't want to be doing *a lot* of complex things in them. It's not a logic running *language* - it's a *helper tool*. So, your average backend developer should be able to handle it well. No highly specialized skills and no dedicated team role for MongoDB queries. 


## Path to the Mongoose Code

Following the path from the demo `listUsers` GraphQL Query schema in `src/relay-queries/user/user-list-query.js`, which defines the resolver, through `src/relay-resolvers/user-resolver.js`, which defines the database query processor as `queryFunction: fetchUserOrderList`, we get to `src/db-handlers/user/user-order-list-fetch.js`, where we find

```
export const fetchUserOrderList = async (
  filterValues,
  aggregateArray,
  viewerLocale,
  fetchParameters
) => {
```

The list of parameters matches the logic coded in the pagination processor `src/paging-processor/find-with-paging.js`. To recap what was explained in the "Paging Flow Solution JavaScript Code" lesson, the following arguments are passed when calling the database query processing function from the pagination wrapper:

- `filterValues` is reserved for a free-form query string that we may be passing from the client; generally, not used in the demo
- `aggregateArray` is the `sort`/`skip`/`limit` component we apply to the query based on the client request and pagination logic implementation
- `viewerLocale` is an element of the `viewer` object that is frequently used when querying the database to extract language-specific subsets of data, e.g., `en` or `fr`. When coding your project, you may replace this with `viewer` or another element of it, based on the use case
- `fetchParameters` are assembled in the query resolver, usually, from the GraphQL query arguments


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

The execution line is wrapped into `try/catch`:

```
  let userArray;
  try {
    userArray = await queryFunc.exec();
  } catch (err) {
    logger.error(`in fetchUserList exec: ` + err);
    throw new Error(`query failed`);
  }
  logger.debug(`  userArray ` + userArray);
```

The result of the query execution is returned to the paging wrapper to apply paging post-processing logic such as *reference point* cursor verification and cursor value generation for each record returned to the GraphQL client.

As an example, for the 4th page of users, sorted by `full_name`, the `queryFunc` will look like this:

```
User.find().sort({full_name: 1}).skip(29).limit(11)
```

The actual query evaluated from `queryFunc` and sent by `mongoose` to the MongoDB engine will be listed in the `Mongoose` debug logging output:

```
Mongoose: user.find({}, { sort: { full_name: 1, username: 1 }, skip: 29, limit: 11, projection: {} })
```


## Error Handling 

The `try/catch` behavior in JS is generally similar to that in Java: an error anywhere inside a chunk of code in the `try` part stops the execution of the chunk and triggers the `catch` part to be run. In NodeJS, however, it looks like the engine sometimes chokes up if an `await` line in the middle of a long `try` section throws and error or *rejects the Promise*. Rather than reaching the catch, the engine may complain about an *unhandled promise rejection*. Particularly, this may be happening if an `await` on a  `mongoose` `exec()` fails. Considering that `await` is more of a syntactical way to *pause* the JS flow waiting for the result than the actual implementation of how async processes are handled by the JS engine, these side effects appear to be plausible. As a precaution, at least with the versions of NodeJS and `mongoose` used in the demo project, each `mongoose` `await` is wrapped in its own `try/catch`. 

In the example above, we *log* the actual error so we can act on it in the log monitor, but we don't return error details to the client. This is the generally recommended practice protecting server internals from exposure to outside systems. In development, GraphQL is configured to run in the debug mode: in GraphiQL, we'll get a detailed error stack leading to the `throw` line. In production, the client will only receive the `query failed` message. The message can obviously be expanded to include some recommended steps, e.g., email address of the support team. In a multi-language system, the `throw` statement would be coded to return a message ID for the translation map vs. a verbal error message.

For a more controlled approach to GraphQL query error handling a designated *query completion code* field can be added to all GraphQL queries return objects. If an error was encountered - its value would be set accordingly. In such a design, all errors in the flow must be *caught* and an empty data reply with the error code set returned instead of throwing an error to the query resolver. The demo project uses this approach when processing Migrations. For Queries, the adopted design relies on the built-in error handling of the GraphQL engine that wraps errors into `errors` object of the response.

The behavior can be tested, e.g., by overriding the `return userArray;` line at the end of `fetchUserOrderList` with something like `return new Error('test error')`


## The DB Query Response

In `mongoose`, data returned by a query can be an array of objects or a single object. If no matching records were found, either an empty JS array `[]` or a `null` is returned. Arguably, for consistency of coding, it would be reasonable to expect from `mongoose` producing uniform replies. Well, let's not bet on this to ever happen - think how much code out there has been written relying on the existing inconsistent output format. As a simple rule, `findOne` and `findById` return *one* record by design, or `null` if nothing was found, whereas `find` or *aggregations* return an array of objects or an empty array.

Looking through debug logging statement in the `node-dev` container console, this is what the database query response is when running `listUsers`:

```
 userArray 
  { primary_locale: 'en', _id: '1E4Yo11Y3r9a', full_name: 'Jane Doe', username: 'jane01',
  primary_email: 'jd@example.com', created_at: <date-time>Z, updated_at: <date-time>Z, __v: 0 },
  { primary_locale: 'en', _id: '1Eg6d1aFL8s7', full_name: 'John Public', username: 'john01',
  primary_email: 'jp@example.com', created_at: <date-time>Z, updated_at: <date-time>Z, __v: 0 }
```

`__v: 0` is a special Document versioning field in `mongoose`. The demo project doesn't use it. The field is included into the database query output when there is no *projection* list specified. Never mind this field. `mongoose` documentation recommends keeping it in the schema (which is the default), so we do.


### Rest of the Logic Flow: from DB Query to Output to the GraphQL Response

For each element of the database query output array, the GraphQL engine will execute the Object Type logic as per `src/relay-models/user-type.js` and populate records of the GraphQL response.

As a note, the GraphQL engine requires that the object returned from the query resolver corresponds to the expected plurality or singularity associated with the output Object Type. In this case, the Object Type is a Connection, so an array is expected. In `getItem` query we'll look at later in this chapter, a single object is expected. So, the fact that `mongoose` queries return either objects or arrays depending on which function is executed - works as an upside here. 

The `edge`, `node` and `cursor` come from the logic coded in `src/paging-processor/connection-from-datasource.js`.

Lastly, you see this `interfaces` definition in `src/relay-models/user-type.js`:

```
interfaces: [NodeInterface]
```

An Interface is an *abstract* type in GraphQL that we don't cover in the course. When you define lots of Types in your application you may find Interfaces convenient to control which fields a group of similar Object Types that *implement* the Interface Type must have. The classic Animal-Cat-Dog example. However, the `interfaces` parameter here has a different meaning. It comes from `graphql-relay` `nodeDefinitions`. You remember that `node` = "the item at the end of the edge". `nodeDefinitions` is a helper object that you can find draft-coded in `src/relay-models/node-definition-types.js`

We don't maintain `nodeDefinitions` in the demo project, but it is included into `graphql-relay` Connections objects that we use. Per the `graphql-relay` design, `nodeDefinitions` should provide default methods of retrieving the content of GraphQL model objects from the underlying dataset using the object's GlobalID. We don't use this kind of stuff in the demo project. All queries have resolvers that implement the dedicated fetching. However, in a more simplistic design, e.g., when navigating over a Graph-like dataset, this direct retrieval of everything by its ID can actually work.


### listUsers Recap and Hungry for More Complexity

`listUsers` looks very simple to implement - and it is. Let's now look at a more complex query that lists User Orders with their Order Items. If you recall from previous chapters, Order Items will act as a *sub-query*, with its *own paging*. Sounds pretty difficult. Well, you'll be disappointed again - it is quite simple, border line primitive, to code. With JS and MongoDB - what else to expect?
