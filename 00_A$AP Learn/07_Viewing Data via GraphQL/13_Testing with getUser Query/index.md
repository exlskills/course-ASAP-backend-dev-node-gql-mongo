### Bonus - getUser Client-defined Query

Let's run this query from `doc/graphql-grahiql-samples.md`, Get User:

```json
query getUser {
  getUser(item_query: "{\"full_name\": \"John Public\"}") {
    id
    username
    full_name
  }
}
```

The schema is in `src/relay-queries/user/get-user-query.js`. `item_query` is a `GraphQLString` argument that we convert into a JS Object and pass into the `src/db-handlers/user/user-fetch.js` `fetchByKey` to use as the filter calling `basicFind` database query wrapper. So, we execute `User.findOne()` `mongoose` function with the exact filter coded on the GraphQL client. As explained before, this can be used in a "trusted expert client" or a "test" mode and should not part of a public API.

Run the query and take a look at these debug log statements:

```
<timestamp> debug:   response: {"primary_locale":"en","_id":"1Eg6d1aFL8s7","full_name":"John Public","username":"john01","primary_email":"jp@example.com","created_at":",<date-time>Z","updated_at":"<date-time>Z","__v":0}
<timestamp> debug:   response obj keys: $__,isNew,errors,_doc,$init
<timestamp> debug:   response obj keys-to: primary_locale,_id,full_name,username,primary_email,created_at,updated_at,__v
```

The debug above comes from this code (see `src/relay-resolvers/user-resolver.js` `resolveGetUser`):

```
  const response = await fetchByKey(queryVal, {}, viewer, info);
  logger.debug(`  response: ` + JSON.stringify(response));
  logger.debug(`  response obj keys: ` + Object.keys(response));
  logger.debug(`  response obj keys-to: ` + Object.keys(response.toObject()));
  return response;
``` 

`response` is the object that came from `mongoose` `findOne()`, and its *keys* (or properties that the object contains) are `$__,isNew,errors,_doc,$init` - not what we'd expect by looking at the debug output of the `response` itself via `JSON.stringify()`:

```
{"primary_locale":"en","_id":"1Eg6d1aFL8s7","full_name":"John Public","username":"john01","primary_email":"jp@example.com","created_at":",<date-time>Z","updated_at":"<date-time>Z","__v":0}
```

Converting the `response` object `toObject()` produces the "correct" keys, as seen on the last debug line: `primary_locale,_id,full_name,username,primary_email,created_at,updated_at,__v`

So, the `mongoose` query result is a *special* object. We hardly ever encounter difficulties processing those objects in regular JS code as most of the time JS functions see the *data content* of it vs. whatever *else* `mongoose` plugs in into there. E.g., `JSON.stringify()` or the GraphQL engine don't require the `toObject()` conversion. 

On rare occasions, however, 3rd party packages may fail processing `mongoose` objects as data. In those cases, use `toObject()`. Just something to keep in the back of your mind when working with `mongoose`. Generally, `mongoose` is helping structuring the application, so, we're keeping it around in the demo project. 


## Use lean() Before exec()

Adding `lean()` before `exec()` in `mongoose` queries turns off the `mongoose` special object functionality. As you probably guessed, the functionality drives `mongoose`-specific model and schema-related stuff such as defaulting and validation. So, without getting into details, if all you need is to *extract* data vs. *retrieve, analyze and update* - `mongoose` objects are useless and can be turned off via `lean()`. 

It is *recommended* using `lean()` on queries that return many Documents to help performance by bypassing `mongoose` object wrapper logic that it applies to each retrieved Document. If you run into performance issues and exhausted optimization on the database side - use `lean()` on large volume queries. Although, you would likely benefit from the mentioned earlier *batching*. So, all things considered, `lean()` may not yield you much on the large schema of things. If your queries return lots of records from the database to the logic server (the one that uses `mongoose`) - you may have a serious design problem in the overall application.


This completes our GraphQL Data Serving chapter! Let's get on to *mutations* - there won't be much new stuff, an easy topic to learn after all the knowledge you've already gained to this point

