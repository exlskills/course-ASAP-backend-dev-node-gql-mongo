### How These Toys Get Deployed to Production

So, we have the environment running locally and we can prove it's working via GraphiQL. Well, everything looks just too simple to be worth a solid production implementation supporting thousands of concurrent users and their orders.

No joke here: a full-functioning Sales Order Management (SOM) system needs a lot more queries and mutations than those coded in the demo project. But they are going to be very similar to what you already have running. Just more of the same, and more *business logic*. Do *not* write business logic in JS, though. Use TypeScript.

Your knee-jerk reaction is to code *serious* stuff in some monolithic app in Java or C#, deploy a Business Process Management (BPM) solution, build a messaging queue to *orchestrate* flows. Without trivializing what a busy IT team has to deal with - think small and *serverless*. Think how to *remove* systems and components vs. what else to *add* to the portfolio.

The two components you're using in the demo - the database and the GraphQL server - will likely stay, no matter what. So, let's figure out how to deploy them into Production.

### Database

Use a reliable service.

### GraphQL Server

Use a reliable, reasonably priced container orchestration service.

### Note On Disabling GrapiQL In Production

Some documentation recommends doing so. As GraphiQL requires the server to do some extra work - this advice may make sense for busy installations. However, that limits your clients from testing things out against your published API. Your choice.
<br>
The actual deployment steps worth going through another course. Not that they require knowledge that an average developer doesn't already have or can't quickly gain. But that *is* another course. So, here we'll just cover a few quick points to help settling the backend development foundation - read on!