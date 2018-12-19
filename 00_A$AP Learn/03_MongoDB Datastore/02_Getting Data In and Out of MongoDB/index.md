### MongoDB Create, Read, Update, Delete (CRUD)

MongoDB comes with a complete set of simple to use CRUD operations. Query conditions for Read and their equivalent "filters" for Update and Delete operations are defined in a form of JSON objects - very convenient for programmatic generation, vs. building text string for SQL. 

MongoDB uses `$`-prefixed command words to form expansive condition statements. 

Each CRUD operation is run against a specific Collection. Other Collections can be linked via *lookup*. 

MongoDB JS API and the proprietary package `mongoose` used in the demo project utilize CRUD syntax and command structure nearly identical to those detailed in the MongoDB documentation. 


### Transactions 

According to the MongoDB documentation, latest versions provide *better* transaction support than initially supplied with the engine. 

As a rule of thumb, the developer should assume that an individual Document is always inserted or updated as a whole (*atomically*). However, as it gets to updating multiple Documents in one or several Collections - the behavior would greatly depend on the DB version and the type of the operation used. As pointed out earlier, MongoDB is best used in business applications that do *not* require transaction-heavy scenarios. 

It is not uncommon using multiple databases in a single application that handle various functionality, e.g., an RDBMS may be employed to handle Product Stock multi-steps commitments and issues, MongoDB - manage supporting documents, and Elasticsearch - enable open-text Product Catalog discovery. Those separate databases would be loosely integrated via near real-time, unblocking replication systems. Only one database would act as the system of record for each particular data category: the Inventory would be in the RDBMS, Product Categories - in MongoDB. Product features and descriptions - in MongoDB as the system of record and also in the Elasticsearch to run text queries against.


### Query Functionality - Find and Aggregations

A basic MongoDB query (and equivalent of SQL SELECT) is executed on a Collection via the `find` command that takes a *filter* argument as well as additional parameters that define return dataset sorting and shaping rules. 

How about something more sophisticated, e.g., similar to a PL-SQL procedure that can execute a multi-step logic directly in the database, without pulling intermediate results into the app? This would be MongoDB *Aggregation*. In its practical form, Aggregation is a series of commands and operations executed as an *aggregation pipeline* - one after another, passing intermediate datasets seamlessly between the steps.

Somewhat unfortunate, *filtering* syntax inside aggregation steps does not always match that in `find`, but, of course, pretty close. There are quite a lot of things that can be *coded* in the pipeline, in addition to filtering, *unwinding* (that we looked at earlier) and sorting. The useful ones we'll see in the demo project

- conditionally including or excluding fields from the result
- renaming fields to drop the dotted notation of embedded objects
- adding dynamically calculated fields to the result
- *joining* Collections via the *$lookup* command 

Aggregations may look quite confusing and hard to debug at first. The good news is that in development and testing, the pipeline can be "stopped" to see the intermediate result - by literally commenting out its remaining tail. So, aggregations *are* difficult to code, especially the first few ones, but do assume that you will *have* to use them working with MongoDB in a professional backend developer capacity.


Will see some interesting examples on using Aggregations in the demo project code. Next - a quick overview of `mongoose`
