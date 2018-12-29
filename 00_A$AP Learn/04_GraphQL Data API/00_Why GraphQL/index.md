### Web Services Rule the App World

Every legacy developer knows about *SOAP* and *REST* API. Most were lucky enough to skip SOAP, but REST has been for a while and still is everywhere these days. *Web Services* is a popular umbrella term covering both SOAP and REST technologies.

Web Services history traces back to the early days of *eCommerce* (another very old and legacy-sounding term) - as a better, more robust alternative to *EDI*. From B2B Systems Integration applications, REST API made a leap into the mainstream Web Development. Who remembers now what "REST" (or "RESTful") stands for? It is quite obvious to everyone these days that a business application should be stateless and asynchronous; clients should communicate with the server over HTTPS protocol on port 443. 

In HTTP, the client sends a *request*, and the server processes the request and returns a *response*.

Aside from *IoT* use cases, which are communications between *devices* or *devices and systems*, most modern business systems integrations and communications are done via Web Services. In modern Web Development, super-popular browser-run *Single Page Applications (SPA)* act as Web Services clients as well. The days when the HTTP *response* was produced as a ready-to-display *HTML page* by the Web Server running something like Java Server Pages and then simply unrolled by the browser - are pretty much over. In today's web apps, the browser (or smartphone) client is the component that constructs web pages - from the *data* that the server provides. Front End discussions are out of scope in this course, let's just say that even if the *pure* SPA model fades away from its current glory, the role of the *Data API* server across both B2B and web development is only going to become more important.

So, with the new Web Development blood coming into the Web Services space, developers realized that REST must be and can be significantly improved - without creating another SOAP-like dinosaur, of course. And here came GraphQL.

### What is GraphQL?

Let's stay focused on the practical application of technology and abstract ourselves from matching names and terms to technology components: which one is a *protocol*, which one is a *framework*, or a *service*, etc. 

Here's the software tooling Use Case: the developer needs a platform to implement and maintain a robust CRUD-style client-server data interface operating via HTTPS. More details:

- the interface should be easy to *set up* and *modify*, self-documenting - for the benefit of both server-side and client-side development teams
- it should enable the client to control the shape of data it needs from a particular *end point* vs. being flooded with the entire dump of whatever dataset has been linked to that end point on the server side
- and, as an added bonus, it should use terminology better fitting data communication use cases than *GET* and *POST*, brought into the app development from the HTTP protocol space

GraphQL has covered the general scope above, and more.

For those who still want a definition of GraphQL as a technology component - [GraphQL creators](https://graphql.org/) say, it is *"a query language for APIs and a runtime for fulfilling those queries with your existing data"*. Great.

### Why Use GraphQL vs. REST?
 
The short answer is: today, you can easily build a much better overall performing and maintainable business solution with GraphQL than with REST.

The learning curve is probably one of the biggest obstacles. Sure enough, as this course shows, you can learn how to use GraphQL on the backend side in a matter of hours. The front end or client side is even less complex, and, incidentally, most of that is covered by the stuff you learn coding the backend.
<br>
So, let's start learning GraphQL without wasting any time!