### Server Code Flow

Now that we have it actually running, let's again read and understand what's in the `src/server.js`. We should use VSC on the host for that.

`startServers()` is the entry point. The `callback` construction allows executing a function after the server's start - in our case, we don't have anything to do, so we don't pass a `callback` to `startServers`. Obviously, this piece came from some old bootstrap project - as we said we're committed to not use *callbacks* in our code. Well, we're not *using* it.

### Connecting to MongoDB

`function startGraphQLServer` starts with the connection to MongoDB. We provide a set of common parameters to `mongoose`, some of which are *workarounds* addressing current state of the product, e.g., `mongoose.set('useCreateIndex', true)` is set by itself vs. in the part where we set `autoReconnect: true` and `useNewUrlParser: true`. As you go through version upgrades, this can be streamlined at some point in the future. The `promiseDb` section is taken from some specific sample and may need to be adjusted in the future as well. This is a normal process of following changes and upgrades in 3rd party npm packages.

### Configuring Express Middleware

`const graphQLApp = express();` - we are using the `express` framework. Our GraphQL stuff runs via `express-graphql`. `express` is a very popular choice when building NodeJS-based web apps, if not the *default*, so we're on board with it.

`express` utilizes the concept of `use` *middleware* and `next()`: the HTTP *request* travels through all the functions added to the flow by `use` statements in the code, in the order they are added. When each function completes successfully, it runs `next()`, otherwise, without `next()` called, the process is assumed ended, and `express` returns whatever HTTP *response* has been put together by that point. Commonly, if the entire flow has not been completed before ending - it means something failed, e.g., the authentication, and an HTTP Error Code is sent back, e.g., `401 Unauthorized`.

Via `graphQLApp.use`, we're adding:

- `/healthcheck` - when the request comes to the `/healthcheck` *path*, e.g., `http://localhost:8080/healthcheck`, the nice little `express-healthcheck` package handles it: "yes, we're up and running". This simple feature is extremely important when deploying production instances, as *Load Balancers* and container orchestration frameworks are designed to  ping running servers continuously (run *healthcheck*), and if no response is received - the server is considered dead and gets replaced

- `cors` (Cross-Origin Resource Sharing) - to enable multi-server handling of clients' requests. E.g., say the client's browser is displaying a `https://exlskills.com` page, so the application running in the browser is considered to be originated from the `exlskills.com` *domain*. If the client application needs to exchange some data with the GraphQL server running on `gql-api.exlskills.com` domain (`exlskills.com` and `gql-api.exlskills.com` are *different* domains), it is supposed to check with the server if the server agrees accepting the request first and only then send/pull the data. The administering of the CORS logic is done by the client's application (usually, at the API level): e.g., it may allow GET requests to always go through and only validate the cross-origin compliance for requests that modify data. CORS is sparsely documented and confusing. The definition of the *origin* consists of the domain as well as the port and protocol. The *setup* is done on the *target* server - which origins the server accepts requests from, by the request's method (e.g., GET, POST, PUT) and any additional applicable conditions. However, the target server by itself is not enforcing the CORS logic, just passes the info to the client app to do so. So, it feels like CORS is designed to protect the client (or User), not the server: if the client side is tempered with, the server has no apparent way of blocking incoming requests on its end.

As a common strategy, for requests that fall under the CORS compliance umbrella, the client's application sends a *preflight* request to the server first. The server replies with a dataset of origins and request types as per its CORS configuration. The client's application checks if what it needs to send is allowed per the dataset - and either sends the actual request to the target server or doesn't. In the latter case, an error may or may not be generated in the browser applications. Without a CORS error, the developer is often left clueless why the request was not sent.

Details of GraphQL API CORS implementation are not scope here. As you watch the browser traffic, you may see two requests going out for each mutation: a preflight one following by the data. Queries appear to be flowing with no regards for CORS validation. As you know, either queries or mutations may be coded to change server data, so the role of CORS in doing any good is quite doubtful. No wonder, most tutorials recommend setting CORS to a `*` - allow any. 

- `cookieParser` - reads *cookies* from the request

- `/graph` - this is the *path* that gets processed by the GraphQL engine. First, `middleware.getViewer` is executed, and if it succeeds, then `graphQLHTTP` kicks in. Notice `middleware.loginRequired` commented out - we'll discuss the access security next.

GraphQL configuration in `graphQLHTTPOpts` is pretty straightforward: enable the `graphiql` browser tester, use `Schema` imported from `schema.js`. 

`context: request.gqlviewer` is the code that tells GraphQL to use `request.gqlviewer` element as the `context`. We've seen `viewer` passed as the third argument to *resolver* functions when we reviewed the GraphQL sample query structure. Technically, the 3rd argument is the `context`, and the configuration above defines that its value comes from the `gqlviewer` element in the `request`. `gqlviewer` is generated in `middleware.getViewer`, so all the ends finally meet.

With the `express` *middleware* flow configured, we can start it listening on the `GRAPHQL_PORT`:

```
graphQLServer = graphQLApp.listen(GRAPHQL_PORT, () => {
```
 
An amazingly simple way configuring and launching an HTTP server process - NodeJS at its best!

So, what server do we run here? Is it NodeJS, or HTTP, or express, or GraphQL? Does it really matter? You can call it *NodeJS* when identifying the backbone technology, or *GraphQL* when taking about the app component in the stack. *HTTP* or *express* will not say much to anyone if used describing this application.
<br>
Next, a quick look at JWT and the `viewer` concept, as those are pretty important in the overall architecture once a server like this goes live
