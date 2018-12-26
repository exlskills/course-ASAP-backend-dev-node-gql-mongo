### Fetching User Order Items from MongoDB 

As outlined earlier, the User Order-level and Order Item-level data is extracted via separate query resolvers and separate database query processor functions. 


## Retrieving Order-level Fields Via `find`

Although, in the demo project, Order and related Order Item are located in one MongoDB Document, *projection* enables us pulling selected parts of the document only. 

In `src/db-handlers/user/user-order-list-fetch.js`, the `find` statement is:

```
let queryFunc = UserOrder.find({ user_id: fetchParameters.user_id }).select({
    order_items: 0
  });
```

Compare to `src/relay-queries/user/user-list-query.js`, there are has two additions:

- filter `{ user_id: fetchParameters.user_id }` inside `find()`
- `select({order_items: 0})` after `find()` part

The filter value `fetchParameters.user_id` is passed from the GraphQL query argument `user_id` into the `fetchParameters` object in the resolver `src/relay-resolvers/user-order-resolver.js`, after conversion from the Global ID and pre-validating against the User Collection:

```
  const user_id_db = fromGlobalId(args.user_id).id;
  const user_rec = await fetchById(user_id_db, { _id: 1 });
  if (!user_rec || !user_rec._id) {
    return Promise.reject('User not found');
  }
  const fetchParameters = { user_id: user_id_db };
```

Note that the pre-validation is done by calling an imported function vs. using `mongoose` data layer functionality directly in the resolver. If `mongoose` is out, the resolver stays intact.

MongoDB `select` can be used to either *include* (`1`) or *exclude* (`0`) fields from the query output. In this case, `order_items` are excluded.


## Fetching Order Items via MongoDB Aggregation Query

`src/db-handlers/order/order-item-list-fetch.js` demonstrates how *aggregation* is used in MongoDB queries. You've been warned in the "MongoDB Datastore" chapter that Aggregation is a complex and advanced functionality, somewhat on par with, e.g, PL SQL. Luckily, the *80-20* paradigm works well when utilizing Aggregation for backend development with MongoDB: 80% of the functionality can be covered with 20% of methods. Even more like *90-10*.

`fetchOrderItemList` well illustrates the Aggregation approach, syntax and covers its key methods:

- aggregation a is *pipeline*: a step-by-step transformation of data, from the raw Collection to the formatted query output. Each step is coded as an element of the *aggregation array*. In `fetchOrderItemList`, an empty array is initialized along with a variable used to build array elements: 

  ```
  let elem,
    agrArray = [];
  ```

  `try/catch` blocks are placed around the array building steps and, separately, the `exec` call (as explained in "MongoDB Query Processor" lesson in this chapter)
- per the MongoDB aggregation syntax, all command / key words inside statements are prefixed with a `$`. Mongo schema field names on the *right side* of statements are also prefixed with a `$`.
- for testing and development, the pipeline can stopped at any point by commenting out the rest of it, up to but not including the `exec()`. The logging statement after `exec()` will print the intermediate output
- a pipeline usually starts with a `$match` command that acts as the filter to limit the number of Documents coming into the pipeline
- `$project` is used to control which fields are needed in subsequent steps, however, the engine is supposed to automatically bring in all fields used in the coded logic, as well as ignore those that don't affect the final output. So, intermediate `$project` statements, if any, are mostly used for streamlining field names, consolidating fields or adding calculated ones. The final `$project` step is the only one required to limit what comes out of the query
- `$unwind` is used to *flatten* embedded arrays to work with the data as a set of separate uniform records. E.g., in `fetchOrderItemList` we `$unwind` the Order Items before we *join* the Item Collection to pull in details required in the final output
- `$lookup` is an equivalent of SQL *join*. In the latest versions of MongoDB, `$lookup` allows executing a *sub pipeline* on the external Collection to control the join conditions and the shape of data coming in


Let's take a closer look at each pipeline step and their intermediate output. 