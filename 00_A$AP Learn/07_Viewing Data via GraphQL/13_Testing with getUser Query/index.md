### Bonus - getUser Free Form Query

As a quick added bonus - let's run this query from `doc/graphql-grahiql-samples.md`:

```json
query getUser {
  getUser(item_query: "{\"full_name\": \"John Public\"}") {
    id
    username
    full_name
  }
}
```

The schema is in `src/relay-queries/user/get-user-query.js`. `item_query` is a free-form `GraphQLString` parameter that we pass, after converting into a JS Object, into the `src/db-handlers/user/user-fetch.js` `fetchByKey` that invokes `basicFind` with `isOne: true` parameter. In other words, we execute `User.findOne()` `mongoose` function passing the MongoDB filter condition directly from GraphiQL. As mentioned before, this is rather an "trusted expert" or "test" mode you would not open up in a public API.

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

`response` is the object that came from `mongoose` `findOne()`, and its *keys* (or parameters that the object contains) are `$__,isNew,errors,_doc,$init` - not what we think they are by looking at the debug output of the `response` itself (via `JSON.stringify()`):

```
{"primary_locale":"en","_id":"1Eg6d1aFL8s7","full_name":"John Public","username":"john01","primary_email":"jp@example.com","created_at":",<date-time>Z","updated_at":"<date-time>Z","__v":0}
```

However, if we convert the `response` object `toObject()` - we get the "correct" keys, as the last debug line shows: `primary_locale,_id,full_name,username,primary_email,created_at,updated_at,__v`

We hardly ever encounter difficulties processing `mongoose` objects as most of the time it fits properly into what JS functions expect, e.g., `JSON.stringify()` or the GraphQL engine. In rare occasions, some external packages may fail to recognize `mongoose` objects as containing the data we know they have. In those (rare) cases, `mongoose` objects should be converted via `toObject()` and then passed into the packages' functions. Just something to keep in the back of your mind when working with `mongoose`. Generally, `mongoose` objects are good for the processing, so we shouldn't just bluntly "turn them off"


This completes our GraphQL Data Serving chapter! Let's get *mutations* reviewed - there will be very little new stuff, so it should be an easy topic to learn at this point

