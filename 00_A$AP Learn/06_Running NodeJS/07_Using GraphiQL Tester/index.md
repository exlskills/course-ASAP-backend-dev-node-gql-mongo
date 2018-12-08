### How to Use GraphiQL Without Confusion

GraphiQL is a browser-based GraphQL testing tool. if you worked with REST API, you almost certainly used [Postman](https://www.getpostman.com/) or ... [SoapUI](https://www.soapui.org/). There are still lots of endpoints out there in the world that those tools can hit, but, luckily for GraphQL devs, the server itself comes equipped with a tester, and it is browser-based. All you need is the URL.

A quick note, that in the JWT scenario, all you need to use GraphiQL from the browser is the JWT token in that browser, which can be obtained via a normal login scenario, first, hitting the authentication server from the browser and logging in.

As we recently tried, going to `http://localhost:8080/graph` from your host browser, opens up the GraphiQL page. Read the brief built-in overview of how to use it - in the left pane. Basically, the left pane is where you enter your queries and mutations, so once you're done reading the overview, you should clear the pane. All the way on top right, there is the `Docs` link that leads you through the entire schema. Note that when the schema changes as you work with the server, you should reload the GraphiQL page for it to get the changes. On reload, GraphiQL preserves everything you typed in (which can be a lot). 

As you type things in the left pane, you'll notice that the URL gets immediately updated. One way to save and share GraphiQL tests is by copying the long-long URL in its entirety and then pasting back into the browser to reuse. Another way is to save the writings in the left pane as text. This is how the content of `doc/graphql-grahiql-samples.md` in the project was built - first in GraphiQL, then "prettified" and copied into this text file. For testing, we'll paste back into GraphiQL, but there are still a few important things to mention before we get to that point:

- GraphiQL requires the syntax to be very precise. It helps highlighting sections it has issues with, but its suggestions and syntax checking engine requires some getting used to, especially when you have lots of single and double quotes to deal with. Exercise patience. Try removing and adding different things to see how it reacts to the variations, and you'll get the syntax right
- The `Query Variables` pane can be expanded by dragging its border up. This is where you put blocks of data you can reference in the query/mutation pane. Mutations in particular rely on variables in the lower section
- You can enter and keep lots of queries and mutations in the left pane and execute those you need only. Just keep in mind GraphiQL does a real-time validation of the entire set, so you can't run anything if something in the pane is wrong 


Ok, now on to trying the GraphQL server in action - getting the data queried. Lots of interesting stuff to see!