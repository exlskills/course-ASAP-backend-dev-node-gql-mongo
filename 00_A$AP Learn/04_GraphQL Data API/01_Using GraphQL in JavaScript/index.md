### Choosing the GraphQL Implementation Platform

We've seen that GraphQL is a *language* and a *runtime* - how can we use it?

In JS, no-brainer, we need a package or few. How about the package named `graphql`?

There are two mainstream implementations of GraphQL on top of the base `graphql` package:

- [relay](https://www.npmjs.com/package/graphql-relay) maintained by Facebook
- [apollo](https://www.npmjs.com/package/apollo-server) maintained by Apollo

As with many technologies, different vendors and development teams build up features on top of the common standard foundation, so the above implementations are compatible when using core GraphQL but differ when it gets to some specific Use Cases. Comparing the two is out of scope of this course. Once you get the basics, fulfilling details will be easy.

The demo project implements a GraphQL backend built with *relay-compatible* features required by the [Relay JS Framework](https://facebook.github.io/relay/). On the Front End side, Relay framework brings powerful functionality that smooths the user experience over slow connections by caching data and managing requests. Respectively, on the server side, Relay requires adherence to its naming convention and coverage of functionality that aids the Front End. 
<br>
A seasoned developer knows that tool selection is a tough process. So, brushing aside numerous posts and marketing pages, let's get into the code!