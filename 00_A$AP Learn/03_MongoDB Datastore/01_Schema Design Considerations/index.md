### Is Document Structure Important?

MongoDB does *not* require Documents to adhere to a specific model, although it has facilities to define Data Models and enforce validation. From the practical standpoint, indexing is the main force behind the need for structure definition, because indexes are build over the structure: you must provide specific field names when creating an index. 

Indexes are literally maps between data and Documents. Documents that don't contain fields used in the index are still included in the index map. MongoDB allows creating `partial` indexes that map only Documents meeting some criteria. This allows keeping indexes on fields that are present (or relevant) in some Documents only concise and efficient.

The relaxed Document structure removes such a huge nuisance of RDBMS as *migrations*, when the entire database has to be physically altered when structure changes, sometimes, taken offline. In MongoDB, if you add more business fields to your logical data model, you may decide that only new Documents should contain those fields - no need for migration or any other action, although, you obviously won't see old data when querying for values in the new fields. Likewise, if you stop maintaining some fields - no need to remove them from historical data. Just this feature alone can be worth switching off an RDBMS.

So, overall the flexibility seems to be a lesser downside than enforcing the model consistency in the DB layer - validate the data in your App, before it gets passed into the DB. As the App is your ultimate producer and consumer of the data - should the logic change, you only have one system to re-code.


### Document Object ID

Each Document in MongoDB must have a unique field `_id`. The engine will assign one automatically when *creating* (inserting) a Document if it is *not* explicitly passed. If the `_id` is passed in, the engine checks its uniqueness across existing Collection Documents and fails the operation if the `_id` is already used. 

The auto-generated MongoDB `ObjectId` type is 12 bytes in size, with the leading bytes representing Timestamp. There are no restrictions on custom `_id` except its uniqueness, enforced by the default unique index on `_id`. Theoretically, different Documents in the Collection may have the `_id` fields of different types of data, but let's not go there: either use the recommended `ObjectId` type or build your own consistent assignment method.

Once the Document is created, its `_id` cannot be changed.


## Embedded Doc ID

An `_id` field can also be put on embedded documents (or anywhere in the doc). MongoDB will *not* treat it in any special way - think of it a regular field. The demo project model contains `_id` on some embedded documents - those used for the business model consistency and assigned via the `mongoose` functionality.
 

### Date and Time

Ideally, the Dates should be stored and handled in queries and filters as MongoDB `ISODate` objects, which happens automatically when the JS *Date* type is used in the JS code and `mongoose` schemas. MongoDB handles Dates internally as numbers, and Dates get automatically converted according to the Time Zone on the way in and out.

The rule of thumb is that as long as you handle Date objects properly in the app code, mindful of the Time Zone values, MongoDB will handle your data correctly. Specifying the Time Zone explicitly takes ambiguity out of the process, but you can also rely on your JS engine defaulting the Time Zone from the runtime environment. 

As an example, let's assume that your browser client, the app server and the MongoDB server are all located in different Time Zones. The client sends a complete Date object to the app server (technically, a numeric value), with the client's Time Zone correctly reflected in it. The app server passes the object as-is into the MongoDB, which stores it. Now, as we fetch the object back into the app server, it gets automatically converted into the corresponding Time Zone of the app server, different clock-wise, but matching the moment of the client's time that came in. JS will take care of the proper conversion, and MongoDB will assure saving the value relatively to the Time Zone it came in marked with. On the other hand, if you mishandle or butcher the client's date interpretation in the app, you may end up passing the client's clock object to the DB marked with the app's Time Zone - get a wrong value stored in MongoDB. In other words, if you use proper Date objects within the flow, modern JS and MongoDB by default will handle the conversions for you on the fly. If you are unsure what objects are used - you can force an explicit conversion of everything into, e.g., UTC, whenever you pass *and retrieve* the data in the app server.

You should also pass explicit Time Zones with your queries to ensure you're filtering the data unambiguously. If you don't - the app's Time Zone will be defaulted in.


### What About Replication and Sharding?

Your database must always be available and operational, which is enabled by continuously replicating data across distributed physical locations. You should still perform periodic backups, mostly as an insurance in case someone or something wipes out your data, but in terms of the hardware availability - it runs 24x7. How do you achieve that? By using a reputable database service for your production. For MongoDB, [Atlas](https://www.mongodb.com/cloud/atlas) is a robust and cost-effective option, especially if your App is hosted at a major Cloud provider (which it should be).

If you use Atlas, you don't need to worry about installing and maintaining your DB - ever.

Sharding allows you controlling how your data is placed. If you got a lot of data, it becomes important where specific subsets of the data are located relatively to providers and consumers. Out of scope for this demo course.


### More on Indexing

Indexes in MongoDB are used to assist search and sorting - no surprise there. 

If a field is inside an embedded *array*, e.g., `"items.sku"`, each element of the array is indexed transparently - as expected. Unlike RDBMS, MongoDB specifically distinguishes between *Single Field* and *Compound* Indexes - those that are built over two or more fields. In RDBMS, the query engine is supposed to figure out the best access path based on all indexes available in the schema. According to the documentation, MongoDB can also do *Index Intersection* - use multiple existing indexes to assist in selecting Documents matching a multi-condition query.

E.g., if selecting some Orders of one User, an RDBMS-trained dev would expect that the engine will use the User index first, then the index on the Order condition, say, Date, to get the result. Not necessarily so in MongoDB: it may end up using one single-field index only, e.g., on the User ID, and then scan over all Orders for that User, even if an index covering the Order filter condition exists. The practical solution is to check all important queries via the Compass "Explain Plan" option to ensure a proper access path used. Conveniently, indexes can be added or deleted on the fly via Compass as well. So, if you want to streamline your multi-condition query, you may need to create a *Compound* index, e.g, on User ID and Order Date in our example. In production, MongoDB Atlas service will send you warnings if your queries cause multi-record scans. Depending on the tier of your service at Atlas, you can even get detailed analysis reports, but usually a quick look at your queries and check via Compass would lead you to the fix.

The direction of indexes - Ascending or Descending - can be specified at the field level, and it is also affects whether or not the index is used processing the query, especially with Compound indexes supporting sort criteria.

Text indexes is an interesting topic, outside of scope of this demo. MongoDB has developed a language-specific text search functionality, which is quite extensive. The best (and only) way figuring it out is by trying different combinations of indexing and search methods. Up to you if you want to give it a shot or move to using something like [Elasticsearch](https://www.elastic.co/) as your open text search solution.


Now that we've covered key schema design considerations - let's look at the CRUD (Create, Read, Update, Delete) principles of MongoDB