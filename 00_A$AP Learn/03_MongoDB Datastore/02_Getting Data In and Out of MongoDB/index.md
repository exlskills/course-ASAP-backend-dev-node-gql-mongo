### MongoDB Create, Read, Update, Delete (CRUD)

MongoDB comes with a complete set of simple to use CRUD operations. Query conditions for Read and their equivalent "filters" for Update and Delete operations are defined in a form of JSON objects - very convenient for programmatic generation, vs. building text string for SQL. 

MongoDB uses `$`-prefixed command words to form expansive condition statements. 

Each CRUD operation is run against a specific Collection. Other Collections can be linked via *lookup*. 

MongoDB JS API and the proprietary package `mongoose` used in the demo project utilize CRUD syntax and command structure nearly identical to those detailed in the MongoDB documentation. 


### Transactions 

According to the MongoDB documentation, latest versions provide *better* transaction support than initially supplied with the engine. 

As a rule of thumb, the developer should assume that an individual Document is always inserted or updated as a whole (*atomically*). However, as it gets to updating multiple Documents in one or several Collections - the behavior would greatly depend on the DB version and the type of the operation used. As pointed out earlier, MongoDB is best used in business applications that do *not* require transaction-heavy scenarios. 

It is not uncommon using multiple databases in a single application that handle various functionality, e.g., an RDBMS may be employed to handle Product Stock multi-steps commitments and issues, MongoDB - manage supporting documents, and Elasticsearch - enable open-text Product Catalog discovery. Those separate databases would be loosely integrated via near real-time, unblocking replication systems. Only one database would act as the system of record for each particular data category: the Inventory would be in the RDBMS, Product Categories - in MongoDB. Product features and descriptions - in MongoDB as the system of record and also in the Elasticsearch to run text queries against.


### Advanced Query Functionality - Aggregations

In addition to a single-statement form of queries via the `find` command and its variations, MongoDB provides an  advanced mechanism extracting the data in a series of commands and operations executed as an *aggregation pipeline*. In practice, Aggregations, once understood by the dev, seem like a much cleaner and more powerful way to approach Query writing than the single statement form. Although, the single statement form enables pipeline-like options as well, e.g., sorting, limiting the number of records returned, *projecting* - specifying which fields to include into the result, Aggregations are practically unlimited in what can be done with the data. Aggregations is the method to *link* data from multiple Collections, effectively filter out irrelevant parts of the Documents or add calculated fields to the output, with the use of conditional statements and variables. 

Aggregations may look quite confusing and hard to debug at first - but as the process in the Aggregation is done in discrete steps - when writing or debugging, results of each step can be printed out and reviewed before writing the next step.


Will see some interesting examples on using Aggregations in the demo code. Next - a quick overview of `mongoose`
