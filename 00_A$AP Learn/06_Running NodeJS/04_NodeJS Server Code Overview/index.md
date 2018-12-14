### Server Code Flow

Let's read and understand what's in `src/server.js` now that we have it actually running. We should use VSC on the host for that.

`startServers();` is the entry point. The `callback` construction allows executing a function after the server's start - in our case, we don't have anything to do, so we don't pass a `callback` to `startServers`.

## Connecting to MongoDB

`function startGraphQLServer` starts with the connection to MongoDB. We provide a set of common parameters to `mongoose`, some of which are *workaround* addressing current state of the product, e.g., `mongoose.set('useCreateIndex', true);` is set by itself vs. in the part where we set `autoReconnect: true` and `useNewUrlParser: true`. As you go through version upgrades, this can be streamlined at some point in the future. The `promiseDb` section is taken from some specific sample and may need to be adjusted in the future as well. This is a normal process of following changes and upgrades in 3rd party npm packages.

## Configuring Express

`const graphQLApp = express();` - we are using the `express` framework. Our GraphQL stuff runs via `express-graphql`. `express` is a very popular choice when building NodeJS-based web apps, if not the *default*, so we're on board with it.

`express` uses the concept of `use` *middleware* and `next`: the HTTP *request* travels through all the functions added to the flow by `use` statements, in the order they are added, and when the function completes successfully, it runs `next()`, otherwise, without `next()` called, the process is assumed ended, and `express` returns whatever HTTP *response* has been put together by that point. Commonly, if the entire flow has not been completed before ending - it means something failed, e.g., the authentication, and an HTTP Error Code is sent back, e.g., 401 Unauthorized.

Via `graphQLApp.use`, we're adding:

- `/healthcheck` - when the request comes to the `/healthcheck` *path*, e.g., `http://localhost:8080/healthcheck`, the nice little `express-healthcheck` package will handle it: "yes, we're up and running". This simple feature is extremely important when deploying production instances, as Load Balancers and Container Orchestration frameworks are designed to be checking status of running servers continuously (run healthcheck), and if no response is received - the server will be restarted.  
- `cors` (Cross-origin resource sharing) - to enable multi-server handling of clients' requests. E.g., say the client's browser is displaying the `https://exlskills.com` page, so it is on the `exlskills.com` *domain*. If the client needs to access the GraphQL server running on `gql-api.exlskills.com` to get some course data, the GraphQL server is *supposed to reject* this request because it is coming from a domain *different* than the one GraphQL server is running on: `exlskills.com` *is not equal* `gql-api.exlskills.com`. This is kind of ridiculous and annoying for devs, but makes some sense in the large schema of things (maybe). Anyway, on the GraphQL server, we can easily whitelist domains we allow such requests from. Note, this is different from blocking or whitelisting the *initial* access to our server: CORS applies to use cases when work with one domain is already ongoing, and then a different domain is being accessed, which gets detected as a cross-origin situation
- `cookieParser` - reads cookies from the request
- `/graph` - this is the *path* that gets processed by the GraphQL engine. First, `middleware.getViewer` is executed, and if it succeeds, then `graphQLHTTP` kicks in. Notice `middleware.loginRequired` commented out - we'll discuss the access security next.

GraphQL configuration in `graphQLHTTPOpts` is pretty straightforward: enable the `graphiql` browser tester, use `Schema` imported from `schema.js`. 

`context: request.gqlviewer` is the code that tells GraphQL to use `request.gqlviewer` element as the `context`. We've seen `viewer` passed as a third parameter to *resolver* functions when we reviewed the GraphQL sample query structure. Technically, the 3rd parameter is the `context`, and the configuration above defines that its value comes from the `gqlviewer` parameter in the `request`. The `gqlviewer` parameter is generated in `middleware.getViewer`, so all the ends finally meet.

With the `express` *middleware* flow configured, we can start the listening on the `GRAPHQL_PORT`:
```
graphQLServer = graphQLApp.listen(GRAPHQL_PORT, () => {
```
 
An amazingly simple way configuring and launching an HTTP server process - NodeJS at its best!

Next, a quick extra look at JWT and the `viewer` concept, as those are pretty important in the overall architecture once a server like this goes live
