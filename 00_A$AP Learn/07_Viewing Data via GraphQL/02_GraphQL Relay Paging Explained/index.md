### Terminology, Theory and Practice

If you've learned about Connections, Edges, Nodes, Cursors in some Data Science class, you already know what they are; read this section to see how they are used in GraphQL.

If you haven't - you've got to map all these terms to what you already know, have seen and used in your practice, especially if you worked with REST API.

*Pagination* is pretty much the default way of passing data via API. Devs often forget about this, because internally, working with data sets, the paging is seamlessly done by the data layer, and the code rarely uses pages when looping through data. In API, assume you always have to write explicit page processing code, so less complex the pagination works - easier it is to write such a code. 

Let's admit, in GraphQL pagination is more complex than in a common REST scenario - because it is designed to do much more advanced things than just flip through pages of data. So, here we've got all these terms that come from a high science and therefore are hard to grasp for someone used to basics only. Thankfully, the practical explanation can be simplified for purpose of writing good GraphQL code. Actually, `node_modules/graphql-relay/lib/connection/connection.js` has everything laid out for us:

- `connection`: "a connection to a list of items". Basically, this is your entire dataset, e.g., the List of User. We used term `connection` when defining Object Types suitable for *pagination*. By doing that, we triggered GraphQL to treat queries that return those Object Types as *paging* queries, so that is why our actual query `listUsers` returns some weird-looking stuff vs. just a simple list fitting a page, with a minimal extra info
- `edges`: "a list of edges"; `edge`: "an edge in a connection". As we put `edges` at the top of the `listUsers` query and got our User records back inside the `edges` parameter, sounds like an edge is our data item , e.g., User. Pretty close
- in the definition of `edge`, `node`: "the item at the end of the edge". Why at the *end*? Never mind, it is *the* item. At the *end*, because the item is kind of *pointed* to
- also in the definition of `edge`, `cursor`: "a cursor for use in pagination". This will make sense very soon, as we look at how the pagination works and what `forwardConnectionArgs` and `backwardConnectionArgs` also defined in `connection.js` represent
- finally, `PageInfo`: "information about pagination in a connection"; `startCursor`: "when paginating backwards, the cursor to continue"; `endCursor`: "when paginating forwards, the cursor to continue"

Here's the added complexity already: we're allowed to page forward (like we usually do in REST) or *back*. Backward paging is a real deal, so the fact that REST doesn't normally do it is *limitation*. As GraphQL is designed not just for machine-to-machine like REST, but for support of human-browser-server flows, it covers the paging to its complete extend.

Along the same lines,  `forward/backwardConnectionArgs` have two parameters each:

- `after/before` - this is the *page size*: how many records do want to show
- `first/last` - this is the tangible *cursor* we want to start the next page at
 
So, we can *dynamically* request, with each query submission, where we want to start, which direction to go, and how many records to return. This is pretty sophisticated.

Our test 2-user dataset can't illustrate much, but it shows what a real would do, by looking at the result we got back in the right pane of the browser:

- we've got the `edges` wrapper that separates the data from `pageInfo`. Inside, for each User, we received the `node` section with all the real User data fields and a weird-looking `cursor` object. If we head over to `https://www.base64decode.org/` and *decode* those two cursor values, we'll get:
  * first cursor: `bW9uZ286MUU0WW8xMVkzcjlhfDB8MA==` decoded as `mongo:1E4Yo11Y3r9a|0|0`
  * second cursor: `bW9uZ286MUVnNmQxYUZMOHM3fDB8MQ==` decoded as `mongo:1Eg6d1aFL8s7|0|1`
  Those 12-pos IDs in the decoded values are the `_id` of the users, so this looks like it has something to do with our MongoDB Documents. In the next section, we'll briefly review what process we use to facilitate the GraphQL paging over our MongoDB collections
- if this was a long sorted list of some business-meaningful info that we review in a browser, we could have selected a record close to the bottom and request to show next 20 records *following* the one we selected, so instead of scrolling and counting, we could have just tell the system to show what we want. Similarly, we could have selected a record and request 50 records *above* it - paging *back*. The `cursor` of the record we selected would be the unambiguous way to tell GraphQL exactly what that record is. In a way, this is similar to the `GlobalID` we've seen used in `graphql-relay`, which was encoded from the *name* of the object type followed by its ID. This is just more generic and technical, as the cursor here always links to the database


As we gained this momentum now understanding what this paging stuff is about, let's take a quick look at the paging engine we use - the one responsible for coding the `cursor` values prefixed with `mongo:`
