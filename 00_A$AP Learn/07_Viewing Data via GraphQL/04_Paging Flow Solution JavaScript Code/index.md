### JS Code Implementing Smart Paging

Folder `src/paging-processor` contains the set of functions that map the paging parameters we've seen in `node_modules/graphql-relay/lib/connection/connection.js` to a *generic function launcher* that executes individual MongoDB queries written to extract specific datasets.

### MongoDB Query Launcher

`src/paging-processor/find-with-paging.js` is the component where the database query function gets called:

```
    return await execDetails.queryFunction(
      args.filterValues,
      aggregateArray,
      viewerLocale,
      execDetails.fetchParameters
    );
```

Function `execDetails.queryFunction` is called with the arguments listed. `execDetails` is an object that we define in the GraphQL paging query schema *resolver* function. E.g., the `listUsers` resolver in `src/relay-resolvers/user-resolver.js`:

```
const businessKey = '_id';
const fetchParameters = {};

const execDetails = {
    queryFunction: fetchUserList,
    businessKey: businessKey,
    fetchParameters: fetchParameters
  };
```

`business key` points to a field in the underlying MongoDB document that is used to uniquely identify the database *record* linked to the *cursor*. The value of that field is loaded into the cursor - we saw that in the previous lesson. In the context of the `lisUsers` query, the business key is the `_id` field of the User Collection. An `_id` would be used as the business key often, but not necessarily always - there maybe other fields in the MongoDB schema that are *unique* relatively to what the query returns, especially if the return dataset doesn't directly correspond to a MongoDB Document. Remember, we added `_id` to some embedded MongoDB fields as well - ensuring that we have unique identifiers at all levels that may be used as datasets. Back to what we discussed in the previous lesson - a generic GraphQL-MongoDB solution can surely be built over a database schema structure fitting exactly each GraphQL query, but it would impose severe limitations on the database design. Pretty much, the MongoDB schema would need to be very dry and resemble a *Graph database*. We don't learn about Graph databases in this course. Relational and NoSQL cover enough application use case ground for a backend developer.

Back to the arguments, `queryFunction` is what runs the MongoDB query. Here, it is `fetchUserList` imported in this line:

```
import { fetchUserList } from '../db-handlers/user/user-list-fetch';
```

The `fetchParameters` argument contains the filtering info to be passed to the database query. Generally, it is assembled in the query resolver from the GraphQL query arguments. As in this query we list all users, no filtering applies.

Back to the `execDetails.queryFunction` call, we also pass:

- `args.filterValues` directly from the query. This argument is included mostly for system-to-system type of interfaces; it is not a good practice opening direct query access to users, so this parameter is ignored in most DB query builders in the demo project
- `aggregateArray` - the evaluated combination of `sort` and `limit` per the GraphQL query, as well as `skip` based on the paging logic

### Utility Methods

`src/paging-processor/aggregate-array-builder.js` and `src/paging-processor/connection-from-datasource.js` contain utility methods executing the logic that assigns the `aggregateArray` parameter values and carries out the cursor-related work. Most of the code is written in a traditional *if-then-else* format that is easy to understand and follow through. There is nothing special in there, just working through the steps we detailed in the previous lesson.

That concludes the smart paging solution overview. Now as you see it action in the GrapiQL results and console logging, you know exactly what it does, how and why.
<br>
Next, will look at the GraphQL Input Type schemas defined in `src/relay-queries/input-types-get-query.js`. Those are used to manage common query input structures on the client side and their mapping into what the server receives