### What is GraphQL?

Every legacy dev knows about SOAP and REST API. Most were lucky enough to skip SOAP, but REST is everywhere these days. Another term used as an umbrella definition over both SOAP and REST is *Web Services*.

Web Services history traces back to the early days of *eCommerce* (another very old and legacy-sounding term) as a more robust alternative to *EDI*. From the Systems Integration use cases, REST API made a leap into the mainstream Web Development. Who remembers what "REST" or "RESTful" stand for? It is obvious to everyone these days that a business application should be stateless and asynchronous, and that the client should communicate with the server over the HTTPS protocol on standard ports. The client asks for data or sends data as a *request*, and the server processes the request and returns a *reply*.

Leaving *IoT* use cases out of scope as those are generally communications between *devices* or devices and systems, business systems integrations and communications are pretty well covered by Web Services and REST. In the traditional web development, though, the *reply* used to be a web page, ready for the client (browser) to display. As it always happens with advances in technology, devs caught on to the point that the same Web Services server, sending strictly REST-like data replies vs. web pages, can also be used in web development, if the web client is sophisticated enough to build a web page around the data by itself. 

With the new blood coming into the Web Services space, devs realized that REST can be significantly improved, without creating another SOAP-like monster, of course. Here came GraphQL.

Let's stay focused on the practical application of technology and abstract ourselves from matching names to tech components: which one is a *protocol*, which one is a *framework*, or a *service*, etc. As a dev, I need a platform to develop and maintain a robust CRUD-style client-server interface operating via HTTPS. The interface should be easy to *set up* and *change*, self-documenting for the next server-side dev coming on board as well as for the dev teams working on the client side. I need to give some freedom to the clients to choose what subsets of data they want to receive from a particular *end point* vs. dumping whatever is coded to be sent when a request comes to that end point. And, as an added bonus, I want to use terminology better fitting my use case than *GET* and *POST*, brought into the app dev space from the protocol's space.

Not sure *mutation* would be my preference over *POST*, but GraphQL has covered the general scope, and more.

In the end, we still have to match GraphQL to a tech component. The [creators](https://graphql.org/) say, it is *"a query language for APIs and a runtime for fulfilling those queries with your existing data"*. Great.

### Why Use GraphQL vs. REST?
 
The short answer is: *today*, you can easily build a much better overall performing and maintainable business solution with GraphQL than with REST.

The learning curve is probably one of the biggest obstacles. Sure enough, as this course shows, you can learn how to use GraphQL on the backend side in a matter of hours. The front end or client side is even less complex, and, incidentally, most of it is covered by what you learn coding the backend.

So, let's start learning, without wasting any time!