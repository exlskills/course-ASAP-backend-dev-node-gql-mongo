### MongoDB NoSQL Storage Model 

Any dev knows what a Host, Database, Table, Row, Column, Field, Index are. MongoDB has Hosts and Databases. In place of Tables, MongoDB has Collections, which are objects to store multiple "Rows", called "Documents" in MongoDB. So, MongoDB is a database that manages storage and retrieval of Documents. Documents have Fields. MongoDB also has Indexes (or `indices` if you prefer).

MongoDB is closely connected to JSON. If you've seen and understand JSON documents - you already know a lot about MongoDB. Technically, MongoDB's storage model is `BSON` or `binary` JSON, but for all practical reasons you can view a document as a JSON-like object, which consists of name-value pairs or arrays of pairs, and each `value` in turn may represent an object or an array. 

One fundamental thing to understand when designing models and working with MongoDB is that when you *retrieve* data, the database engine works to locate *documents* vs. blocks of global JSON-formatted data. Say, you convert an Order into one JSON document, where the ordered Items would be placed into an array, similar to:
```json
{ 
  "orderNumber": 123,
  "delverToUser": 456,
  "billToUser": 456,
   "items":[
      {"sku": 1001,
       "qty": 5},
      {"sku": 2001,
       "qty": 10},
      {"sku": 3001,
       "qty": 15},
   ],
   "deliveryBy": "2020-01-01"
}
```
All Orders are stored into the Order *Collection* in your MongoDB Database. After some time, the Collection would contain a bunch of documents in it (we hope, the selling business is going well). Now, if you need to extract all orders for Item with sku=2001, you would need to query the Order Collection, and by definition you'd get back all Order documents that contain this Item, not just an extract of some JSON elements from here in there in your Collection. Certainly, you can have the engine to further process the documents and build an exact dataset that you need from the documents located (the key - still do that inside the DB engine, before sending the extracted data back to the app server over the network wire). But those dataset *reducing* operations are done as manipulations on given JSON objects vs. on an indexed structure. In other words, if you build an index on `"items.sku"` in your Order Collection, MongoDB will use the index and instantly find all orders that contain items with sku=2001. Then, if you just want to see the ordering information for that item, you need to ask the engine, for each Order document, to *unwind* (flatten) the array of "items", which will make the document looking like this:
```json
{ "intermediateDoc" :
  [
  {"orderNumber": 123,
  "delverToUser": 456,
  "billToUser": 456,
  "items": {"sku": 1001,"qty": 5},
  "deliveryBy": "2020-01-01"
  },
  {"orderNumber": 123,
  "delverToUser": 456,
  "billToUser": 456,
  "items": {"sku": 2001,"qty": 10},
  "deliveryBy": "2020-01-01"
  },
  {"orderNumber": 123,
  "delverToUser": 456,
  "billToUser": 456,
  "items": {"sku": 3001,"qty": 15},
  "deliveryBy": "2020-01-01"
  },
  ]
}
```
Then filter the intermediate document to include only elements with "items.sku"=2001 and return the information we need from the order itself, e.g.,
```json
{ "resultDoc" :
  [
  {"orderNumber": 123,
  "qty": 10,
  "deliveryBy": "2020-01-01"
  }
  ]
}
```

What does that mean for the dev? Design your data model in such a way that you'd be able to extract information you often need without having the engine to dig inside of Collection-level documents for it. Again, by building an index on an *embedded* element (in our example, "items" are embedded into the Order document) you can efficiently retrieve all the documents from the Collection that contain the embedded value, however, retrieving a subset of embedded information from each document would require additional processing in the engine. Naturally, if you have just a few Item lines per Order, storing an Order as a single document makes sense when you business model operate mostly at the Order level, and even when you need to extract a subset of Items - the performance penalty is not critical. However, if your business model is Item-focused and your Orders contain many Items - you should store one Order Line per document in a separate OrderItem Collection. In the latter case you may decide to copy some Order information that doesn't change into each OrderItem collection Document and/or use the `lookup` functionality to link your OrderItem and Order Collections when retrieving data only available in the Order Collection.

And of course, the ultimate goal is, as always in the App-Database design, to minimize the amount of data traveling from the DB into the App. No filtering in the App - always do it in the database.

### Is Document Structure Important?

MongoDB does not require documents to adhere to a specific model, although it has facilities to define Data Models and enforce validation. From the practical standpoint, indexing is the main force behind the need for structure definition, because indexes are build over the structure: you must provide specific field names when creating an index. As indexes are literally maps between data and documents, documents with absent fields may still be present in the map of Collection's indexes on those fields. To control that MongoDB allows creating `partial` indexes that map only documents meeting some criteria. This allows keeping indexes on fields that are present (or relevant) on some documents only concise and efficient.

The relaxed structure requirements take away such a huge nuisance of RDBMS as *migrations*, when the entire database has to be physically altered on any structure change, sometimes, taken offline. In MongoDB, if you add more business fields to your logical data model, you may decide that only new documents would contain those fields - no need for migration or any other action, although, you obviously won't see old data when querying for values in the new fields. Likewise, if you stop maintaining some fields - no need to remove them from historical data.

So, overall the flexibility seems to be a lesser risk and downside than enforcing the model consistency in the DB layer - validate the data in your App, before it gets passed into the DB. As the App should be your ultimate producer and consumer of the data - you would have to re-code your logic in one system if needed so.

### Object ID

Each Document in MongoDB must have a unique field `_id`. The engine will assign one for you when creating a Document, unless it is explicitly provided (in which case the engine checks its uniqueness and rejects the doc if the `_id` is already present in the Collection). The built-in MongoDB `ObjectId` type is 12 bytes in size where the leading bytes represent the Timestamp. If you'd rather use your own value that is unique and have a business meaning - you can, just need to pass it in when creating the Document. Once the Document is created, its `_id` cannot be changed.

There is a default index on `_id`, so querying and filtering by `_id` is always fast.

### Date and Time

Ideally, the Dates should be stored and handled in queries and filters as MongoDB `ISODate` objects, which happens automatically when the Date field type is used in JS and `mongoose`. As the Date is handled internally as a number and converted according to the runtime Time Zone (or the explicitly provided one) when used as a Date, the Date object always corresponds to a specific moment in time, although, numerically different across Time Zones.

The rule of thumb is that as long as you handle Date objects properly in the code, maintaining the Time Zone values, MongoDB will store your data correctly. Specifying the Time Zone explicitly takes ambiguity out of the process, but you can also rely on your JS engine defaulting the Time Zone based on the given runtime environment's clock. Just make sure you don't incidentally override the time somewhere along the processing path. 

As an example, let's assume that your client, app server and MongoDB server are all in different Time Zones. The client sends a complete Date object to the app server (technically, a numeric value), with the client's Time Zone correctly reflected in it. The app server passes the object as-is into the MongoDB, which stores it. Now, as we fetch the object back into the app server, it gets automatically converted into the corresponding time of the app server, different clock-wise, but matching the moment of the client's time that came in. JS will take care of the proper conversion, and MongoDB will assure saving the value relatively to the Time Zone. On the other hand, if you mishandle or butcher the client's date interpretation in the app, you may end up passing the client's clock object to the DB, but with the app's Time Zone in it. In other words, if you use proper Date objects within the flow, modern JS and MongoDB by default will handle the conversions for you on the fly. If you are unsure what objects are used - you can force an explicit conversion of everything to, e.g., UTC, whenever you pass *and retrieve* the data in the app server.

You should also pass explicit Time Zones with your queries to ensure you're filtering the data unambiguously. 

### What About Replication and Sharding?

Your database must always be available and operational, which is achieved by continuously replicating data across distributed physical locations. You still do periodic backups, mostly in case someone or something wipes out your data, but in terms of hardware availability - 24x7. How do you achieve that? By using a reputable database service for your production. For MongoDB, [Atlas](https://www.mongodb.com/cloud/atlas) is a robust and cost-effective option, especially if your App is hosted at a major Cloud provider (which it should be).

If you use Atlas, you don't need to worry about installing and maintaining your DB - ever.

Sharding allows you controlling how your data is placed - if you got so much of it that it starts really matter for the overall performance of your solution where specific subsets of data are located relatively to your data providers and consumers. Out of scope of this demo course.

### More on Indexing

Indexes in MongoDB are used to assist search and sorting - no surprise here. If a field is inside an embedded array, e.g., `"items.sku"`, each element of the array is indexed transparently - as expected. Unlike RDBMS, MongoDB specifically distinguishes between *Single Field* and *Compound* Indexes - those that are built over two or more fields. In RDBMS, the query engine is supposed to figure out the best access path based on all present indexes to choose from. According to the documentation, MongoDB can also do *Index Intersection* - use multiple available indexes to assist in selecting Documents matching a multi-condition query. E.g., if selecting some Orders of one User, an RDBMS-trained dev would expect that the engine will use the User index first, then the index on the Order condition, say, Date, to get the result. Not necessarily in MongoDB: it may end up using one single-field index only, e.g., on the User ID, and then scan over all orders for that User, even if the another index covering the Order filter condition exists. The practical solution is to check all important queries via the Compass "Explain Plan" option and see the access path used. Conveniently, indexes can be added or deleted on the fly via Compass as well. So, if you want to streamline your multi-condition query, you may need to create a compound index, e.g, on User ID and Order Date in our example. In production, MongoDB Atlas service will send you warnings if your queries cause multi-record scans. Depending on the tier of your service at Atlas, you can even get detailed analysis reports, but usually a quick look at your queries and check via Compass would lead you to the fix.

The direction of indexes - Ascending or Descending - can be specified at the field level, and it is also affects whether or not the index is used processing the query, especially for Compound indexes.

Text indexes is an interesting topic, outside of scope of this demo. MongoDB has developed a language-specific text search functionality, which is quite extensive and can only be figured out by trying different combinations of indexing and searching methods. Up to you if you want to give it a try or move to using Elasticsearch as your open text search solution.