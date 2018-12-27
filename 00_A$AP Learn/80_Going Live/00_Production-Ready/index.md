### How These Toys Get Deployed to Production

So, we have the environment running locally and we can see it working from GraphiQL. Everything looks just too simple to be worth a solid production implementation open to thousands of concurrent users actually entering some meaningful orders.

No joke here: a full-functioning Sales Order Management (SOM) system needs a lot more queries and mutations than those in the demo project. But they are going to be very similar to what you have running. Just more of the same, and more *business logic*. Do *not* write business logic in JS. Use Type Script.

Your knee-jerk reaction is to code *serious* stuff in some monolithic app in Java or C#, deploy a Business Process Management (BPM) solution, build a messaging queue to *orchestrate* flows. Without trivializing what a busy IT team has to deal with - think small and *serverless*. Think how to *remove* systems and components vs. what else to *add* to the portfolio.

The two components you're using in the demo - the database and the GraphQL server - will likely stay, no matter what. So, let's figure out how to deploy them into Production.


## Database

Use a reliable service.


## GraphQL Server

Use a container orchestration service.


The actual deployment steps worth another course. Not that they require knowledge that an average developer doesn't already have or can't quickly gain. But that *is* another course. So, here we'll just answer the questions to help settling the backend development foundation - read on!