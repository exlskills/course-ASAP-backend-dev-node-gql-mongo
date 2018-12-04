### MongoDB Create, Read, Update, Delete (CRUD)

MongoDB comes with a complete set of simple to use CRUD operations. Query conditions for Read and their equivalent "filters" for Update and Delete operations are provided in a form of JSON objects. MongoDB uses `$`-prefixed command words to form more complex statements. The Collection's name is specified at the root of the statement, before the actual statement. 

JS API and its proprietary version `mongoose` used in the demo project utilize a very similar syntax and the command structure nearly identical to those detailed in the MongoDB documentation. 

### Transactions 

According to MongoDB documentation, latest versions provide better transaction support. Generally, the dev should assume that an individual document is always inserted or updated as a whole, however, as it gets to updating multiple documents in one or multiple Collections - the behavior would greatly depend on the DB version and the operation used. As pointed out earlier, MongoDB is best used in business applications that don't strain transaction-heavy scenarios. It is not uncommon using multiple databases in a single application that handle various functionality, e.g., an RDBMS may handle multi-steps product stack commitments and issues, MongoDB - manage supporting documents and Elasticsearch - provide open-text product catalog discovery. Those separate databases would be loosely integrated via near real-time, unblocking reconciliation system, but only one database would act as a system of record for a particular data category: the inventory would be in the RDBMS, product categories - in MongoDB, as well as features and descriptions, *indexed* (loaded) into both MongoDB as the system of record and Elasticsearch to run queries against.

### Advanced Query Functionality - Aggregations

In addition to a single-statement form of queries, via `find` command and its variations, MongoDB provides a pretty advanced way of extracting the data in a series of commands and operations executed as an *aggregation pipeline*. In practice, Aggregations, once understood by the dev, seem like a much cleaner and more powerful way to approach Query writing than the single statement form. Although, the single statement form enables pipeline-like options as well, e.g., sorting, limiting the number of records returned, *projecting* - specifying which fields to include into the result, Aggregations are practically unlimited in what can be done with the data. Aggregations is the method to *link* data from multiple Collections, effectively filter out irrelevant parts of the Documents or add calculated fields to the output, with the use of conditional statements and variables. 

Aggregations may look quite confusing and hard to debug at first - but as the process in the Aggregation is done in discrete steps - when writing or debugging, results of each step can be printed out and reviewed before writing the next step.

Will see some interesting examples on using Aggregations in the demo code.
