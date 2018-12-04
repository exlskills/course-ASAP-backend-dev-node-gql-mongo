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


### What About Replication and Sharding?

Your database must always be available and operational, which is achieved by continuously replicating data across distributed physical locations. You still do periodic backups, mostly in case someone or something wipes out your data, but in terms of hardware availability - 24x7. How do you achieve that? By using a reputable database service for your production. For MongoDB, [Atlas](https://www.mongodb.com/cloud/atlas) is a robust and cost-effective option, especially if your App is hosted at a major Cloud provider (which it should be).

If you use Atlas, you don't need to worry about installing and maintaining your DB - ever.

Sharding allows you controlling how your data is placed - if you got so much of it that it starts really matter for the overall performance of your solution where specific subsets of data are located relatively to your data providers and consumers. Out of scope of this demo course.