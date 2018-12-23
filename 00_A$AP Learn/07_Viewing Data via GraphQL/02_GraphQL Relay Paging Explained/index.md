### GraphQL Terminology, Theory and Practice

If you've learned about Connections, Edges, Nodes, Cursors in a Data Science class, you already know what they are - browse through this section to see how they are used in GraphQL. If you haven't learned Data Science - let's map these terms to those that you already know and used in your IT practice, especially if you worked with REST API.

Developers often forget that *pagination* should be used by default when passing data via API. When we work with datasets inside an application, the paging is seamlessly done by the data layer, and the code rarely uses pages or batches explicitly when looping through data. In data exchange API, assume you always have to write explicit page processing code, so the less complex it is to get pagination working - the easier developing the application. GraphQL and Relay take pagination use cases to the next level compare to a common REST scenario of exchanging data *page by page*. The functionality is being described by terms that come from the high science and may sound weird to an IT practitioner used to working with simple and clear terminology. 

Thankfully, the practical explanation of GraphQL theory and terminology can be quite simple. No need to look far - `node_modules/graphql-relay/lib/connection/connection.js` has everything laid out for us:

- `connection`: "a connection to a list of items". Basically, this represents all records in a particular dataset, e.g., all Users, all Orders, etc. Except, connection is an *intermediary* that is used to get to the data in the list. Fine. In "GraphQL Data API" chapter, you saw the term `connection` used when defining Object Types suitable for *pagination*. By doing that, we signalled to GraphQL to treat queries that return those Object Types as *paging* queries. That is why query `listUsers` returns a bunch of weird-looking stuff vs. just a simple *page* of data. A connection is *all* records, in contrast to the following terms that all define specific subsets
- `edges`: "a list of edges"; `edge`: "an edge in a connection". In the `listUsers` query in GraphiQL, `edges` is at the top of the set of fields we want to get back. In the result that came from the GraphQL server, the User records are inside of the `edges` parameter. So, an edge is one item in the dataset, e.g., a User. Pretty close
- in the definition of `edge`, `node`: "the item at the end of the edge". Why at the *end*? Never mind, it is *the* item. At the *end*, because the item is kind of *pointed* to. The concept of an *intermediary* to the data vs. just the data itself. Ok 
- also in the definition of `edge`, `cursor`: "a cursor for use in pagination". This will make sense very soon, as we look at how the pagination works and what `forwardConnectionArgs` and `backwardConnectionArgs` also defined in `connection.js` represent. So, cursor is an *intermediary* layer-level unique identifier for the item in the dataset
- finally, `PageInfo`: "information about pagination in a connection"; `startCursor`: "when paginating backwards, the cursor to continue"; `endCursor`: "when paginating forwards, the cursor to continue". Very nice - cursors can be used generically as *stakes* inside the dataset. Rather than showing something like the User ID of the last User on the page, GraphQL identify cursors. We also have to use *cursors* when controlling paging or asking for some subset of data. 
 
Now it all comes together: the terminology completely defines the *intermediary* layer that is used by GraphQL  to manage navigation over a dataset generically vs. via some ID inside the dataset that would be dataset-specific and different across various datasets. Well, obviously, there got to be a defined way of getting from the generic *cursor* to some tangible *record id* in the dataset. Yes, exactly. That is what the actual implementation is about. So, for developers, there is not much value in this intermediate layer if a bunch of code still needs to be written to get from this abstraction to the concreteness of the data. We make a mental note of what these terms mean - as we *have to* use them when working with GraphQL - and move on to looking at the real stuff.


## More on GraphQL Controlled Paging

In GraphQL, we're allowed to page forward (like we usually do in REST) or *backward*. Backward paging is a real deal in a human-browser-server application, so the fact that REST doesn't normally do it is a limitation. In addition to controlling the direction, we can also control the starting point of the page of data we want to receive. In `node_modules/graphql-relay/lib/connection/connection.js`, `forwardConnectionArgs` and `backwardConnectionArgs` have two parameters each:

- `after/before` - this is the *page size*: how many records do want to show
- `first/last` - this is the *cursor* (the record in our dataset) we want to start (or, *finish*) the next page at
 
So, in GraphQL, we can *dynamically* request, with each query submission, where we want to start, which direction to go, and how many records to return. This is pretty sophisticated.

Our test two user records dataset can't illustrate much, but it illustrates what a query can do and how the result returned by the server looks like.

Reviewing what we got back in the right pane of the browser, we see that the `edges` wrapper containing the data and the `pageInfo` wrapper with some generic info about the page. Inside `edges`, for each User, we received the `node` section with all the real User data fields and a weird-looking `cursor` object. If we head over to `https://www.base64decode.org/` and *decode* those cursor values, we'll get:

- first cursor: `bW9uZ286MUU0WW8xMVkzcjlhfDB8MA==` decoded as `mongo:1E4Yo11Y3r9a|0|0`
- second cursor: `bW9uZ286MUVnNmQxYUZMOHM3fDB8MQ==` decoded as `mongo:1Eg6d1aFL8s7|0|1`

Those 12-pos IDs in the decoded values are the `_id` of the users, so this looks like it has something to do with our MongoDB Documents. The code that facilitates the GraphQL paging over our MongoDB collections is reviewed in the next section

In a real scenario browsing through a list of some business-meaningful data, we can select, e.g., a record close to the bottom and request to show next 20 records *following* the one we selected. Or, select a record and request 50 records *above* it - paging *backward*. The `cursor` of the record that we select would be passed to GraphQL to define where to start. In a way, the cursor is similar to the `GlobalID` we've seen used in `graphql-relay`. If you recall, `GlobalID` was encoded from the *name* of the object type followed by object's ID. The cursor, in this implementation, in effect, is an encoded link to a database record. Note that GraphQL doesn't impose any standard on the cursor content - completely up to the developer what should go into there.


As we gained this momentum now understanding what this paging stuff is about, let's take a look at the implementation of the paging engine we use - the one responsible for coding the `cursor` values prefixed with `mongo:`. Get ready - the stuff is pretty technical. Well, this is a developer training course, isn't it?
