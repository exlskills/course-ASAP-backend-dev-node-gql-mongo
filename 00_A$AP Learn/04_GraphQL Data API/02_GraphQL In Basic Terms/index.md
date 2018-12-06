### Making GraphQL Simple

GraphQL has a lot of theory behind it, but there are just a few simple practical things to know that will get you started using it as your backend coding platform:

- the purpose of GraphQL to facilitate *request-reply HTTPS* communications. The client's request can be a *query* or a *mutation*, and the server always replies sending some data back
- all structures in scope of the process are well-defined; they become part of the `schema` that we've seen before while looking at the demo project's code components. The client has to follow the defined structure when sending requests, and the result coming back adheres to format expected - no ambiguity
- majority of the structures represent 
  1. a business type with object's fields, e.g., a User Type object
  2. a query with request fields, process function and reply type definition, e.g., ListUsers
  3. a mutation with input payload fields and types, process function and reply type definition, e.g., SendOrder

  although, these structures serve different purposes and align to different syntax standards, they look quite similar - lists of fields and elements
- each object in the structure has a name. When the client sends a query - it passes the name of the query to the server, along with the required (or optional) filtering data, as per the query definition 
- the *schema* is owned by the server and relevant parts of it are shared with the client. In the Relay framework, the client's schema must be explicitly generated on the server and copied into the client's code base. The schema is also accessible at runtime - as the base for a *service discovery* functionality
- from the client's view point, queries are designed to request and receive data, and mutations are designed to send changes to data and receive a feedback. On the server side, both queries and mutations are processed in a similar way. Technically, data can be modified when processing a query, as well as a mutation process may just be sending information back to the client, emulating a query action. To outline this straight: the framework doesn't control *what* the server does, it just *helps* you structuring the read and write flows differently by using queries and mutations
- JS functions that carry out processing logic are specified in the definition of each query and mutation inside the *schema* - on the server side. The client doesn't know how the process is done. The functions must be coded by the server-side developer. The GraphQL framework controls when and how the functions are called, but it doesn't provide any ready to use code for handling a particular datasource. Controversially, there are 3rd party packages that can be configured to bridge GraphQL and, e.g., MongoDB directly. We'll see that a typical query/mutation execution function for GraphQL with MongoDB is trivial to code, but there is still enough logic that may need to be covered specifically vs. via a generic package configuration
- the framework takes care of formatting HTTPS request-reply, the dev would normally won't even see what is in there

### GraphQL Backend Dev Tasks

Now that we know how GraphQL works, here are the tasks the dev should cover building a GraphQL backend in NodeJS:

- write GraphQL Object Types for all business data objects, e.g., User, Item, Order
- define which queries should be implemented, e.g., ListUsers, ListItems, ListUserOrders
- define which write operations should be implemented, e.g., SendUser, SendItem, SendOrder, UpdateOrderItem
- write GraphQL definitions for the above queries and mutations, specifying the data going and out, as well as setting up placeholders for the implementation functions
- write the implementation functions for each query and mutation
- run NodeJS and test everything via `GraphiQL`, which is a browser-based interactive tool to execute GraphQL queries and mutations against your backend
 
### The Confusing Part

So, in the described paradigm, we definitely have pieces that resemble the familiar MVC and REST API puzzles:

- the Models are the in the MongoDB backend
- the schema defines multiple Views that client works with; those are seen in the Object types and filter/payload definitions
- the query/mutation processing functions are the Controllers that broker the Model-View data exchange
- the requests have clear way of addressing "end points" by providing query/mutation names and placeholders to send parameters and payloads

Sounds like everything is covered. Yes, but actually, GraphQL query/mutation processing functions do not put the data directly into the client-facing layer. The Object Types are not just static lists of View-like fields, but rather full-blown mapper engines that ingest the result produced by the processing functions (called *source*) and groom it according to the *functions* associated with each output field. We'll see next, that most of the mapping is "defaulted in" from the *source*, but the mapping capabilities of Types open up many interesting opportunities for devs

### The Added Bonus

Lastly, as pointed out, the client knows upfront the exact structure of the reply to the query. By GraphQL design, the client *must* explicitly send the exact list of fields (and elements, in a multi-layer structure) it wants to receive. And it gets back only what's requested. No more uncontrolled REST-like data dumps

So, this is simple, but requires at least a couple of more rounds of detailing to be really understood. Let's take a look at the actual schema components in the demo project
