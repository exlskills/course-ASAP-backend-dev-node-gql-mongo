### Paging Solution Implementation for MongoDB

As mentioned in "GraphQL Data API" chapter, GraphQL or Relay does not deliver any specific code that facilitates processes at the data layer, so we either have to find a 3rd party package that does that or code our own stuff.

This section talks about some pretty deep implementation details that are not directly related to the GraphQL technology but illustrate the scope of work that must be done by the dev. Read on, although, a word of caution - the details are kind of difficult to grasp from the first try. Do your best, if not clear - move on, it will sink in as you work with the code, just get the idea for now.

There are certainly a selection of JS packages offering GraphQL, and `graphql-relay` in particular, connectivity to MongoDB. Some may look too primitive, assuming a certain way that MongoDB schemas should look like to work well with GraphQL schemas, some are documented badly, so it's hard to figure out what they do, without getting deep into the code.

`src/paging-processor` contains a set of simple functions that map the paging parameters we've just seen in `node_modules/graphql-relay/lib/connection/connection.js` to a generic function launcher that runs custom MongoDB queries that we write.

Arguably, the biggest complexity doing this flexible paging correctly is to account for changes in the underlying dataset that may be taking place while the user session is in process. Basically, if the user repeatedly runs a query showing the next 10 alphabetical Users after John, and we've got lots of people joining in, the list will look differently as new records come in into the database. The same impact would cause our simple forward paging to break if we just assume that the *next* page after the *last* record displayed can be retrieved by *skipping* and *limiting* the same base *database* query to the next record count. Here's the catch - if we can run our database query efficiently, filtering it exactly to what we need to display the *next page* - there is no problem: we run a new query for each page and get exactly what the user asks for, from the latest dataset. Unfortunately, there is no "generic" reliable way to do so, and we don't want to be figuring out for each query we write how it should look like to facilitate the paging. E.g., in one case, we need alphabetical users greater than Jon, in another, we need users sorted by their last sign-in date, etc. We'd have to write lots of similar queries to cover all types of sorting and conditions the system would offer.

So, the practical paging solution would be to use `skip` and `limit` that MongoDB provides. This way, we re-run the same database query and instruct the database engine to return only the slice of records that GraphQL requested, based on paging. Skipping and limiting is relatively cheap on the database side. Now, what happens if our dataset has shifted? Our skipped records in the new database query will not be the same as in the previous one, so if we go solely by counts, we won't be presenting the real "next page" *after* the last cursor on the previous one. So, here's a simple thing we can do - we can check if that *last cursor* is at the position in the *new* database query where we expect it to be. If yes - great, we are skipping correctly, our dataset hasn't changed in the area we've been skipping over. If no - we have to look at the entire dataset, find where our *last cursor* is now, and return the right number of records after it as the *next page*. 

Say, on the server side, we got the first page returned to the user, running some sophisticated database query with sorting and filtering, and `limit` set to the requested page size. The user asks for the next page: we run the same database query, adding the `skip` parameter at the amount of records we returned last time, *minus one*, and `limit` to the page size *plus one* . We *optimistically* expect that the first record in the new set returned by the database query will start with the *last record* we sent last time. If yes - we remove that record from the set and pass the rest to the user. If no - we run the database query wide open and search for where our *last record* is now, then send the page-size amount of records after it to the user. And so on.

This approach also works if the user sent the *cursor* to *start from*, but changed the direction, sorting or filtering since the previous query (which GraphQL allows doing). Rather than analyzing if there was a change - we run our *optimistic* query with the conditions requested. If the conditions changed, we won't find the old *cursor* where we expect it to be, so, we'll run the query again and get the records the user requested. 

So, the last two values in the decoded `cursors` we saw show the location of each cursor in the *entire* dataset that our database query is returned, skipping and limiting *aside*. The first value defines if we count records from the beginning of the dataset or from the end (depends on the user query direction), and the second number shows the position: 

- first cursor: `bW9uZ286MUU0WW8xMVkzcjlhfDB8MA==` decoded as `mongo:1E4Yo11Y3r9a|0|0`
- second cursor: `bW9uZ286MUVnNmQxYUZMOHM3fDB8MQ==` decoded as `mongo:1Eg6d1aFL8s7|0|1`
 
Here, we count from the beginning of the dataset `|0`, and our records are `|0` and `|1`. Say, we returned 10 records in one page (pos 0-9), 10 more in the next (pos 10-19) - the third page cursors will have their dataset positions from 20 to 29. Now, if the user asks for 10 after 25, we'll receive that "25th" cursor with the user's query. To process the request, we run the database query (with the filtering and sorting per user's current request), passing `skip`=24 (something like that, +- one). In the result set that comes from the database, we check the "business key" of the first record and compare it with the one recorded in the "25th" cursor (e.g., the business key value of our record 0 above is `1E4Yo11Y3r9a`). Depending on the match, we continue the logic as described.

Sounds like running the query two times is a bad approach? Well, we only do that if the dataset changed or the user changed the way of paging through the *same* data (the user sent us a record from that data to use as the anchor) - in a typical business scenario this would not be happening a lot. 

## Where The Code Is?

`src/paging-processor/find-with-paging.js` is the core place where the database query function gets called:

```
    return await execDetails.queryFunction(
      args.filterValues,
      aggregateArray,
      viewerLocale,
      execDetails.fetchParameters
    );
```

Here, literally, the function `execDetails.queryFunction` gets called with the parameters listed. `execDetails` is an object that we define in each query *resolver* function. E.g., in `src/relay-resolvers/user-resolver.js`:

```
const businessKey = '_id';
const fetchParameters = {};

const execDetails = {
    queryFunction: fetchUserList,
    businessKey: businessKey,
    fetchParameters: fetchParameters
  };
```

The "business key" of the MongoDB document, in the context of the query, is `_id` (this is the case most of the time, but not necessarily always - there maybe other fields in the MongoDB schema that are *unique* relatively to what the query returns, especially if the return set doesn't directly correspond to a DB Document - where the `_id` is defined).

The function that runs the MongoDB query is `fetchUserList`, and it is imported into this JS file as you can see at the top:

```
import { fetchUserList } from '../db-handlers/user/user-list-fetch';
```

The `fetchParameters` generally link back to the GraphQL query arguments. As in this query we list all users, those are empty by design.

Back to the `execDetails.queryFunction` call, we also pass:

- `args.filterValues` directly from the query (this is included mostly for system-to-system type of interfaces; it is not a good practice opening direct query access to users, so this parameter is ignored in the DB query builders in the demo)
- `aggregateArray` - the evaluated combination of `sort` and `limit` per the user query and `skip` based on where we want the dataset returned by the database query to start


Next, will look at the argument types defined in `src/relay-queries/input-types-get-query.js` to see how to control values in client query input