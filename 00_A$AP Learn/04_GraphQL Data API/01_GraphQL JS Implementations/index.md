### Choosing the GraphQL Implementation Platform

We've seen that GraphQL is a *language* and a *runtime* - how can we use it?

In JS, no-brainer, we need a package or few. How about the package named `graphql`?

As it often happens in technology, different vendors and dev teams take some base foundation and then add on to it in different ways. As GraphQL facilitates what can be a complex client-server async flow, there is lots of room for implementing details of it differently.

Currently, there appear to be two mainstream implementations of GraphQL on top of the base `graphql` package:
- [relay](https://www.npmjs.com/package/graphql-relay) maintained by Facebook
- [apollo](https://www.npmjs.com/package/apollo-server) maintained by Apollo
 
The demo is built with relay-compatible features, which outline the advanced use of paging, driven by the [Relay JS Framework](https://facebook.github.io/relay/). Relay has lots of powerful browser client functionality that smooths the user experience over slower connections by caching the data and managing requests. On the server side, Relay places additional naming convention and feature implementation requirements, which would be optional if the client does not require specific Relay functionality.

A seasoned dev knows that tool selection is a tough process, and it's way tougher without a good knowledge about the tech space that the tool covers. So, leaving aside numerous posts and marketing pages, let's get into the code!