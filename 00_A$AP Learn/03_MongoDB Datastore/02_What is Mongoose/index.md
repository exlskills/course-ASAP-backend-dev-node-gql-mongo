### Data Access Layer

The role of `mongoose` is to act as the data access layer - bridging the JS code and the MongoDB storage layer. In the RDBMS world, the data access layer API is expected to be fast, transparent, neutral to the way the solution is implemented on the either side of it. `mongoose`, on the other hand, is actually viewed as a complementary extension making work with MongoDB more structured and organized vs. just passing the data back and fourth. 

The `mongoose` "intelligence" certainly makes it an additional external dependency point in your solution, so if you prefer more direct control over your data layer - skip it and use something like the [MongoDB Node.JS Driver](https://mongodb.github.io/node-mongodb-native/)

In the latest versions, `mongoose` has caught up with the `async/await` JS coding paradigm.

All operations in `mongoose` are done after a connection to the DB is established (although, due to the *buffering* feature, you can work off-line for some time). Conveniently, `mongoose` keeps the connection *alive* by default, so that the connection doesn't time out. Just like many other DBs, MongoDB does limit the number of active connections: check your Atlas tier specifications (you'll find the limit quite generous). By default, `mongoose` creates a 5-connection pool to the DB for multi-threading. This may potentially affect the I/O if you do frequent partial Document updates very close to each other, but rather hypothetically. 

The Collections are accessed via `mongoose` *Models*, which are built from *Schemas*. The Schema contains all the field-level definitions and properties for the Document, like types, defaults, and much more. Per `mongoose` documentation, Documents are instances of Models, which makes sense on the JS side, although, the necessity of the Schema layer seems questionable: why not just put the definitions directly into the Model? One clue may be that not every Schema is equal to a Model: embedded objects can be defined in a separate Schema and reused in multiple parent Schemas. MongoDB uses the Schema and Model concepts as well, but MongoDB Model is as logical as the Schema, unlike in `mongoose`.

All `mongoose` CRUD operations are executed on a Model. There are Schema-level operations, e.g., to define indexes. So, at a minimum, each Collection being accessed from the app via `mongoose` must be listed as a Model. `mongoose` will help converting data types and defaulting values, including generation of *Created At* and *Updated At* timestamps. Although, Updated At is ignored in some operations. `mongoose` will also validate data adherence to the Schema, unless explicitly bypassed.

To invoke the async mode for each call vs. the `then` mode, use the `exec()` method passing the commands to the DB - you'll see that used throughout the demo code.

In the Debug logging mode, `mongoose` prints out the details of operations and statements - very convenient in the dev process.

We'll look at the common `mongoose` usage scenarios in the latter chapters, starting with the Schemas and Models in the demo project in the next lesson, read on