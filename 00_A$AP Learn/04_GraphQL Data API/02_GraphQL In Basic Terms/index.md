### Keeping GraphQL Simple

GraphQL has a lot of theory behind it, but there are just a few simple practical things to know that will get you started with it as your backend professional coding platform:

- the purpose of GraphQL is to facilitate *request-response HTTPS* communications. The client's request can be a *query* or a *mutation* (=update), and the server always replies sending some data back
- all structures in scope of the process are well-defined in the GraphQL `schema` - remember one that we've seen reviewing demo project code components. The client has to follow the defined structure when sending requests, and the reply coming back adheres to format expected - no ambiguity
- the structures represent: 
  1. an Object Type with a list of fields in it, e.g., a User Type object
  2. a Query that defines the structure of the request, the server function to process the request, and the Type to be sent as the response, e.g., listUsers
  3. a Mutation with input payload fields and Types, the server process function, and the response Type definition, e.g., sendOrder

  although, these structures serve different purposes and align to different syntax standards, they end up looking quite similar to each other
- each object in the schema must be given a name that the object, e.g., a query, is known by to the clients. E.g., when a client sends a query - it passes the name of the query to the server, along with the request data formed per the query structure definition 
- the schema is owned and maintained on the server side. Relevant parts of the schema are shared with the client software. In the Relay framework methodology, the client-visible part - `schema.graphql` file we've seen in the project structure - must be generated on the server and copied into the clients code base. The client / Front End developer references objects from the file in the code being written. 
 The schema is also accessible at runtime - as the base for *service discovery* functionality
- from the client's view point, queries are designed to request and receive data, and mutations - to pass changes to data and receive a feedback. On the server side, both queries and mutations are processed in a similar way. Technically, the server can modify data when processing a query, or query-and-reply when processing a mutation. To outline this straight: the framework doesn't control *what* the server does, it just *helps* you structuring the read and write flows differently by using queries and mutations as separate types
- JS functions that carry out the processing logic are specified directly in the definition of each query and mutation in the server-side schema. The client doesn't know how the process is done - the client side-distributed schema extract doesn't include those. 
- GraphQL framework controls when and how the processing functions are called, but it doesn't provide any ready to use code for handling a particular flow or datasource. So, the functions must be coded by the server-side developer. Controversially, there are 3rd party packages that can be used to bridge GraphQL and, e.g., MongoDB directly - we'll see that a typical query/mutation execution function for GraphQL with MongoDB is pretty trivial to code. In a real implementation, though, there may still be enough custom logic to cover by writing code vs. configuring in a generic external package, so we don't review any of those in the demo project
- the framework takes a complete care of formatting HTTPS request-response. Normally, the developer would never even see the raw HTTP the framework produces

### GraphQL Backend Development Tasks

Now that we know the basics of GraphQL, here is the list of tasks that the developer building a GraphQL backend in NodeJS will be working on (given the business requirements):

- write GraphQL Object Types for all business data objects, e.g., User, Item, Order
- define the scope of queries to be implemented, e.g., listUsers, listItems, listUserOrders
- define the write operations scope for the implementation, e.g., sendUser, sendItem, sendOrder, updateOrderItem
- write GraphQL definitions for the queries and mutations in scope, specifying the data going and out, as well as setting up placeholders for the JS implementation functions
- write the JS implementation functions for each query and mutation - from the request, via the database layer, to the response
- run NodeJS with debug-level logging and test the solution launching queries and mutations via the `GraphiQL` browser-based interactive tool 
 
### Just like MVC and REST - And More

So, in the described paradigm, we definitely see pieces that resemble the familiar MVC and REST API solutions:

- the Models are in the MongoDB backend
- the Views are in the GraphQL schema - the layers that the client works with
- the Controllers are the query/mutation processing functions that broker the Model-View data exchange
- the "end points" for requests are the names of queries and mutations
- the payloads are the values passed into the queries and mutations per the defined schema

As was mentioned before, in GraphQL the client has control to select which parts of the server-offered reply structure does it want to receive back - kind of client-defined Views. To implement this feature, GraphQL makes Object Types "smart" vs. just a "dumb" inventory of fields.

In the GraphQL server-side flow, the Controller-like processing functions *prepare* the data to be sent out, but the Object Types are the components that physically map the data to the client-bound output. Seamlessly for the server-side developer, only the fields and elements that the client explicitly asked for are sent out (this relates to queries - in mutations, the reply sent back is server-controlled, within the structure described in the schema).

We'll review this in great details going forward, but the key takeaway at this point: in GraphQL, the processing functions do not map the output. They prepare an object that becomes the *source* for the Object Types to execute the mapping. So, Object Types, in turn, often contain *functions* that process additional logic, on top of what the core processor has done.

With a little extra work and smart design - no more uncontrolled REST-like data dumps flooding the wires and clients.

### Client-side GraphQL

In this course, we are building a GraphQL server. There are client-side GraphQL components that are not in scope of this course, as well as some difference in terminology that we'll see towards the end of the course.

You know, there is always more to learn out there. "The more you know, the more you know you don't know". So, let's keep the focus, and once you cover the server-side, you'll decide where to go next. Maybe, you'll end up learning more about the client-side GraphQL - maybe not.
<br>
So, GraphQL is simple, but let's continue detailing it so the foundation is well understood. Next, we'll take a look at the actual schema components as those are coded in the demo project
