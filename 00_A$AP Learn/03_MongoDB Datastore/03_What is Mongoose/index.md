### Mongoose As The Data Access Layer

The role of `mongoose` is to act as the data access layer - bridging JS code and the MongoDB storage layer. 

Thinking of something like *JDBC* in the RDBMS world, the data access layer is expected to be fast and transparent - almost invisible. `mongoose`, on the other hand, is actually designed to act as a complementary extension to MongoDB, making it more structured and organized - vs. just passing data back and fourth. 

The `mongoose` "intelligence" certainly makes it a significant additional external dependency for the implemented solution, so for those who prefer sticking with a *thin* data access layer - skip `mongoose` and use [MongoDB Node.JS Driver](https://mongodb.github.io/node-mongodb-native/)

In the latest versions, `mongoose` has caught up with the `async/await` JS coding paradigm, so that is the mode used in the demo project.

### DB Connection 

First thing when running 'mongoose' is establishing the DB connection. Here's surprise #1: due to its *buffering* feature, `mongoose` may start executing pieces of code like queries or updates in the *off-line mode* - before the DB connection is in place. It is one thing to work off-line if an established connection flickers, but start processing data *before* the DB acknowledges and authenticates - sounds crazy. Well, apparently, this is a feature of the MongoDB Node.js Driver. Buffering can be disabled by setting `bufferMaxEntries` to 0 and `bufferCommands` to false in the connection properties. In the demo project, the default buffering is kept - as the app is coded to shut down if the initial `mongoose` connection can't be established. The expectation is that the app won't get too far while the connectivity is pending to worry about this feature.

Conveniently, `mongoose` keeps the connection *alive* by default, so that it doesn't get timed out by the MongoDB server while the app is idle. 

Keep in mind, like many other DBs, MongoDB does limit the number of active connections: check your Atlas tier specifications (you'll find the limits quite generous). By default, `mongoose` creates a 5-connection pool to the DB for multi-threading. Parallel processing may potentially affect the results if you run frequent, very close to each other, partial Document updates and/or query Documents being updated - but rather hypothetically. As mentioned earlier - MongoDB is not your platform to implement transaction-dependant flows.

### DB Schema

MongoDB Collections are accessed via `mongoose` *Models*, which are built from *Schemas*. A Schema contains all field-level definitions and properties of a Document: types, defaults, and much more. 

Per `mongoose` documentation, Documents are JS Objects that are *instances* of `mongoose` Models - makes sense on the JS side. The necessity of the Schema layer, though, seems questionable: why not just put the definitions listed in the Schema directly into the Model? One clue may be that not every Schema is equal to a Model: embedded objects can be defined in a separate Schema and reused in multiple parent Schemas. Also, *Model* in `mongoose` is described as a *constructor* compiled from *Schema*. Whatever.

MongoDB widely uses both Schema and Model concepts as well, but MongoDB Model is not a concrete element like it is in `mongoose` - just a term used when describing design patterns.

Basically, trying to make a lot of sense cross-referencing terminology between MongoDB and `mongoose` is not necessary to use the tool efficiently: you just write some code following a few simple patterns, like those plentiful in the demo project. 

### CRUD

All `mongoose` CRUD operations are executed on a `mongoose` Model (which is MongoDB Collection, see the above). There are Schema-level operations, e.g., to define indexes. So, at a minimum, each Collection being accessed from the app via `mongoose` must be present in the JS code as a `mongoose` Model. `mongoose` will help converting data types and defaulting values, including generation of *Created At* and *Updated At* timestamps. Although, beware, Updated At is left unchanged by some C-U-D operations. `mongoose` will also validate data adherence to the `mongoose` Schema, unless explicitly bypassed.

To invoke the async mode for each call vs. the old style `then` mode, use the `exec()` method to the command to the DB - you'll see examples throughout the demo project code.

### Logging

In the Debug logging mode, `mongoose` prints out details of operations and statements - very convenient for development and troubleshooting.
<br>
We'll look at the common `mongoose` usage scenarios in the latter chapters, starting with review of the demo project Schemas and Models in the next lesson, read on!