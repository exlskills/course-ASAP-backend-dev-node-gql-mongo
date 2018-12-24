### Paging Solution Implementation for MongoDB

As mentioned in "GraphQL Data API" chapter, GraphQL or Relay packages do not include any specific code to facilitate processes at the data layer, so we either have to find a 3rd party package that handles that or code our own stuff.

This section talks about some pretty deep implementation details that are not directly related to the GraphQL technology but illustrate the scope of work that the developer needs to cover when building a solution. Read on. A word of caution - this may be difficult to grasp from the first try. Do your best, if something is not clear - keep on moving: it will sink in as you work with the code, just get what you can for now.


## Finding a 3rd Party Package

There is certainly a selection of open source JS packages available in GitHub that cover connectivity from GraphQL, and `graphql-relay` in particular, to MongoDB. Some may look too restrictive, assuming a certain way how MongoDB schemas should look like to work well with GraphQL schemas. Others are documented badly, so it's hard to figure out what they do without getting deep into the code. As a developer, you know, often times, writing your own code is faster than figuring someone elses out and adopting.


## The Definition of Proper Paging

To implement flexible paging correctly we must account for the impact of possible changes in the underlying dataset while the data review session is in progress. 

As a simple example, if the browsing user repeats a query showing next 10 alphabetical Users *after* the one named John Public, and there are lots of people joining the User list, the result will look differently over time, as new records come in into the database. 

Similarly, changes to the underlying dataset would cause a basic forward paging to break if we assume that the *next* page can be retrieved by running the same query as the one that returned the *current* page and simply *skipping* over more records. 

Here's the catch - if we could form our database query to retrieve the *next page* reliably using some tangible filtering condition vs. *skipping* over records that we *assume* were already displayed - there would be no problem paging through a dynamically changing dataset. However, in a real app use case, there is no simple generic way to do so. E.g., the browsing user may ask for alphabetical users greater than John Public, or for users joined after a particular date, or for usernames greater than John01, etc. If we're to use query-level filtering to page through the User dataset in all those different scenarios, we'd have to either write lots of database queries or code a database query generator to cover for all types of sorting and conditions the system offers. Good luck with that.


## The Smart Skipping Solution

So, a practical paging solution can't rely on ability to use database query-level filtering, but rather utilize skipping over previously returned records, with logic added to *correct the course* if shifts in the underlying data affect the *page* the browsing user wants to see.

We must stay calm and don't get paranoid over changes to the underlying dataset *in general* while paging. You know that each invocation of a GraphQL paging query requires providing the *reference point* cursor, except when asking for the initial page. Those are specified in the `after` or `before` arguments of the `...connectionArgs`. So, as long as we can get to the database record corresponding to the reference point for the particular request - we can retrieve the *actual* page of data the browsing user wants from the real-time content of the database and send it back.

We'll utilize `skip` and `limit` query parameters that MongoDB provides. Skipping and limiting is relatively cheap on the database side as it takes place internally. The database query will be formed based on *non-paging* filtering criteria and sorted the way the browsing user requests. `skip` and `limit` will be applied to the overall result set the query returns - to have the database engine selecting only the slice of records based on requested *paging* arguments values. E.g., the filter would be set on something like `primary_locale="fr"` and the sort order by the `full_name`. The `skip` will reflect the number of pages we already sent out and `limit` - the size of one page. Well, we just said above that this is not going to be correct if the underlying data shifted - we'll either send some records again, or miss if we simply `skip` and `limit`. Let's keep on digging.

What happens if our dataset has shifted? Our actual skipped records in the core database query *may* not be the same in subsequent query executions. So, here's a simple thing we have to do - check that the *reference point cursor* for this database query execution is *at the same position* in the database query result where we *expect* it to be based on the previous execution of the database query. If yes - great, we are skipping correctly, our dataset hasn't changed *in the area* we've been skipping over. If no - we're off and must take a corrective action: re-query to retrieve the entire dataset, find the *reference point cursor* in it and return the right number of records after (or before) it as the *page* requested. 

Let's reiterate this again in an example. The browser user asks for the initial page of data. On the server side, we execute a sophisticated database query with sorting and filtering, and `limit` the set to the requested page size. The user asks for the next page passing the *cursor* of the *last* record returned with page-1. We run the same database query, adding the `skip` parameter at the amount of records we returned last time, *minus one*, and `limit` to the page size *plus one* . We *optimistically* expect that the first record in the set returned by this database query will start with the *last record* we sent last time. If yes - we remove that record from the set and pass the rest to the user. If no - we run the database query wide open and search for where our *last record* is now, then send the page-size amount of *current* records after it to the user. And so on.


## What if the GraphQL Query Content Changed?

So far, we looked at paging through a dataset using *a given* GraphQL query: next, next, next. What happens if the browser user wants to change some filtering conditions or the sort order, still sending a *reference point cursor* with the query. How can we distinguish this changed query request from an unchanged *next page* one? We cannot, because

- on the GraphQL side, a *next page* or a *different page* queries are syntactically formed in exact the same way
- we don't keep any query history by user on the server side.

Is this a problem? Not really. If the query changed significantly, the *reference point cursor* will not be at the same position in the *optimistic* database query result as it was in the previous query result set. Note that we always run the database query using the actual arguments that came in via GraphQL. So, we will have to write off the *optimistic* query result and run the database query again, without skipping / limiting.

What *position in the result set* are we talking about when describing the *reference point cursor*? Very simple. Assume, there 100 records in the entire dataset that fall under the filtering condition, and they are sorted according to the sorting condition provided in the GraphQL query. Here we go - we can assign a number to each of those records, e.g, from 0 to 100. As we physically extract the records and send them to the GraphQL client in pages, the *cursor* that we attach by design to each record contains that # from 0 to 100. So, if the *reference point cursor* is at the position 88 in our set - we can add something like `skip: 87` to the database query - and get right to that record. Assuming, of course, that the database query is exactly the same as the one that *previously extracted* that record as being the 88-th record in the filtered dataset. If the database query is different - we'll get that record returned at a different position or not at all. 

So, the last two values in the *decoded* `cursors` that we saw in the right pane of GrapiQL show the location of each cursor in the dataset that the database query returns. The first value defines if we count records from the beginning of the dataset or from the end (depends on the user query direction), and the second number shows the position: 

- first cursor: `bW9uZ286MUU0WW8xMVkzcjlhfDB8MA==` decoded as `mongo:1E4Yo11Y3r9a|0|0`
- second cursor: `bW9uZ286MUVnNmQxYUZMOHM3fDB8MQ==` decoded as `mongo:1Eg6d1aFL8s7|0|1`
 
Here, we count from the beginning of the dataset `|0`, and our records are `|0` and `|1`. Say, we returned 10 records in one page (pos 0-9), 10 more in the next (pos 10-19) - the third page cursors will have their dataset positions from 20 to 29. Now, if the user asks for 10 after 25, we'll receive that "25th" cursor with the user's query. To process the request, we run the database query with the filtering and sorting per user's current request, passing `skip`=24 (something like that, +- one). In the result set that comes from the database, we'll check the "business key" of the first record and compare it with the one recorded in the "25th" cursor (e.g., the business key value of our record 0 above is `1E4Yo11Y3r9a`). Depending on the match, we continue the logic as described.

Why do we have to define the record's position in the filtered dataset as two numbers vs. just one? Because we *do not know* how many records are in the *entire* filtered dataset as we're paging through it. We only know about the records that we have retrieved. And depending on the sorting of the dataset and the direction of viewing (forward vs. backward) - sometimes, we may have to process the dataset from the *end* vs. *start*. Just a technicality. The 1st number is a *flag*: `0` when we go from the start and `1` - from the end of the dataset. The second # is the *position*.

Sounds like running the query two times is a bad approach? Well, we only do that if the relevant part of the dataset changed or the user changed the way of paging through the *same* data (the user sent us a record from that data to use as the reference point cursor) - in a typical business scenario this would not be happening a lot.


Let's see how the code doing the above looks like