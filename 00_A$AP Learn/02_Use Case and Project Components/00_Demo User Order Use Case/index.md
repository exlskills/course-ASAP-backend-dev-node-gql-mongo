### The Course Sample Use Case 

A backend use case usually includes a front end (FE) piece as means to invoke the functionality. Luckily, the *GraphQL API* that we employ as the backbone, comes with a built-in data access facility, so we can develop a complete solution and view it in action without having to code or borrow a FE app. 

Though, wouldn't it make more sense building a backend and a FE together? After all, one of the benefits of writing the backend in JS is the alleged code reusability between the two "ends" of the overall solution. This course is deliberately backend-only. Modern FE technologies are complex and specialized frameworks. Choosing, learning and understanding such a technology is a task that requires quite a lot of preparation and background knowledge. Nowadays, FE frameworks evolve so fast that 12-15 month old code is hardly maintainable or upgradable anymore. Doesn't sound like a field to venture into in your first legacy-to-modern retooling course. Don't worry, there is enough work in the backend, and it is often considered a harder skill to gain - for IT newcomers, those without the professional foundation that you have.

 Once you master the modern backend and get more familiar with JS and related techs, getting how the FE works will be much easier. If after completing this course you decide that "full stack" is where your dev career should go, with the knowledge gained here, you'll feel much more comfortable reviewing FE frameworks and choosing one that makes more sense to you as a professional vs. following an "advise" from someone fresh out of a coding camp.


## So, What is The Use Case We Are Going to Code?

Sales Order Entry: Customers, Items, Orders. We will call Customers *"Users"* - as modern applications are always self-service oriented. 

Here's the business process flow in scope: the User gets access to the system, picks Items, creates an Order; the Order is stored in the database and can be retrieved via the API.


Next, we'll start working with the actual code!