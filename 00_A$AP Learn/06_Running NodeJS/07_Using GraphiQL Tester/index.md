### How to Use GraphiQL Without Too Much Confusion

GraphiQL is a browser-based GraphQL testing tool. if you worked with REST API, you almost certainly used [Postman](https://www.getpostman.com/) or ... [SoapUI](https://www.soapui.org/). There are still lots of endpoints out there in the world that those tools can hit, but, luckily for GraphQL developers, the server itself comes equipped with a tester, and it is browser-based. All you need is the URL. 

A quick note, when JWT logic is uncommented and required, before running GraphiQL, you need to login into the actual web application that uses your GraphQL server. The browser will receive the JWT token from your authentication endpoint in a cookie associated with the GraphQL server domain. Then you can run GraphiQL from that browser - the JWT will be automatically sent with the cookie.

As we briefly tried in "Starting GraphQL Server in NodeJS" lesson, going to `http://localhost:8080/graph` from your host browser opens up the GraphiQL page. Read the brief built-in overview on how to use it that pops up in the left pane. Basically, the left pane is where you enter your queries and mutations, so once you're done reading the overview, you should clear it out. 

At the top right, the `Docs` link leads you through the entire GraphQL schema of the server. Note that when you modify the schema you should reload the GraphiQL page for it to read in the updated version. Don't worry, on reloads and even browser restarts (don't clear the history), GraphiQL preserves everything you typed in, which can be a lot. 

As you type things in in the left pane, you'll notice that the URL gets immediately updated. Here's the common way saving and sharing GraphiQL tests: copy the entire super-long URL. It can be pasted back into another browser to reuse the testing setup. Another way of saving and sharing tests is via copy/paste of writings in the left pane as text. This is how the content of `doc/graphql-grahiql-samples.md` in the project was built - first in GraphiQL, then "prettified" and copied into the text file. 

The leading part of the URL is what defines the target GraphQL server - very simple. Need to test against a different server - change a few characters in the URL.

A few important things to mention before you get to pasting of provided demo tests into you browser:

- GraphiQL requires the syntax to be precise before a query or mutation can be sent to the server. It highlights sections it has issues with, but its suggestion and validation engine requires some getting used to, especially when you have lots of single and double quotes to deal with. Exercise patience. Try removing and adding different characters to see how it reacts to the variations, and you'll get the syntax right
- The `Query Variables` pane (bottom left) can be expanded by dragging its horizontal border. This pane is used to enter data that can be referenced in the query/mutation pane to keep it neat and generic. Mutations in particular rely on variables put into this pane. JSON format is used
- You can enter and keep lots of queries and mutations in the left pane and execute one at a time as need. Just keep in mind that GraphiQL does a real-time validation of the entire set, so you can't run anything if something in the pane is wrong 


Ok, now on to trying the GraphQL server in action. Will start with querying data. Lots of interesting stuff to see!