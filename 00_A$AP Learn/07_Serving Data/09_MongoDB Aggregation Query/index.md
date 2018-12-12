### Fetching User Order Items from MongoDB 

As outlined earlier, the User Order-level and Order Item-level data is extracted via separate query resolvers, which in turn, logically, use separate database query processors. 

## Retrieving Order-level fields via `find`

Although in MongoDB order and item data are combined in a single Document, *projection* enables us pulling selected fields of the document only. In `src/db-handlers/user/user-order-list-fetch.js`, the `find` statement has two additions compare to what we saw in `src/relay-queries/user/user-list-query.js`:

```
let queryFunc = UserOrder.find({ user_id: fetchParameters.user_id }).select({
    order_items: 0
  });
```

- filter `{ user_id: fetchParameters.user_id }` inside `find` itself
- and `select({order_items: 0})` after `find`

The filter value `fetchParameters.user_id` is passed from the GraphQL query argument `user_id` into the `fetchParameters` object in the resolver `src/relay-resolvers/user-order-resolver.js`, after convertion from the Global ID and pre-validating against the User Collection:

```
  const user_id_db = fromGlobalId(args.user_id).id;
  const user_rec = await fetchById(user_id_db, { _id: 1 });
  if (!user_rec || !user_rec._id) {
    return Promise.reject('User not found');
  }
  const fetchParameters = { user_id: user_id_db };
```

Note, that the pre-validation is done by calling an imported function vs. using `mongoose` data layer functionality directly in the resolver.

The `select` can be used to either list the fields to include into the `mongoose` query output (marking them with a `1`) or exclude (`0`). In this case, `order_items` are excluded


## Fetching Order Items via MongoDB Aggregation Query

`src/db-handlers/order/order-item-list-fetch.js` demonstrates how *aggregation* is used in MongoDB queries. You've been warned in the "MongoDB Datastore" chapter that Aggregation is as huge of a topic and complex as advanced SQL Query and something like PL SQL combined. Luckily, the *80-20* paradigm works well for backend development with MongoDB Aggregation: 80% of functionality can be covered with 20% of commands. Even more like *90-10*.

`fetchOrderItemList` well illustrates the approach, syntax and covers key methods used in MongoDB Aggregation:

- aggregation is *pipeline*: a step-by-step transformation of data, where each step is represented as an element of the aggregation array. In `fetchOrderItemList`, an empty array is initialized along with a variable used to build its elements: 

  ```
  let elem,
    agrArray = [];
  ```

  Additionally, `try/catch` blocks are placed around the array building steps and, separately, the `exec` call (as explained in "MongoDB Query Processor" lesson in this chapter)
- per MongoDB aggregation syntax, all command / key words inside statements are prefixed with `$`. Mongo schema field names on the *right side* of the assignment statements are also prefixed with the `$`.
- for testing and development, the pipeline can stopped at any point by commenting out the rest of it, up to but not including the `exec`. The logging statement after `exec` will print the intermediate output
- a pipeline usually starts with a `$match` command that acts as a filter to limit the number of Documents coming into the pipeline right from the beginning
- `$project` is used to control which fields are needed in the subsequent steps, however, the engine would pull in missing fields if required as well as remove those that don't affect the final output. So, in reality, intermediate `$project` statements, if any, are used for streamlining field names, consolidating multiple or adding calculated fields, and the final `$project` is the only one required to limit what comes out
- `$unwind` is necessary to *flatten* embedded arrays to work with their elements in subsequent steps at the individual record level. E.g., in `fetchOrderItemList` we `$unwind` the Order Items before we *join* the Item Collection to pull in details required in the final output
- `$lookup` is the equivalent of SQL *join*. In the latest versions of MongoDB `$lookup` is designed nicely to allow expansive filtering options on how the join works, by executing a *sub pipeline* on the external Collection


Let's take a closer look at each pipeline step and their intermediate output. 