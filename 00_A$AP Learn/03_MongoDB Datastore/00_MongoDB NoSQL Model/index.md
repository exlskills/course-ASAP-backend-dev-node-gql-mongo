### MongoDB NoSQL Model 

Any professional developer knows what a Host, Database, Table, Record, Column, Field, Cell, Index are. Those who spent their careers working with Relational databases will find MongoDB very familiar and recognizable, in spite of its *NoSql* designation. 

MongoDB has Hosts and Databases. For Tables, MongoDB uses *Collections*, which are objects to store multiple Records, called *Documents* in MongoDB. So, MongoDB is a database that manages storage and retrieval of Documents. Documents have *Fields* with *Values*. MongoDB also has *Indexes* (or `indices` if you prefer). Check, check, check - everything is covered.

MongoDB is deeply rooted in JSON. If you understand JSON documents - you already know a lot about MongoDB. Technically, MongoDB storage model is `BSON` or *binary* JSON, but for all practical means you can view a MongoDB Document as a JSON-like object, which consists of name-value pairs, or arrays of pairs. A Document's field value may represent an object or an array - here we have objects *embedded* into Documents. That is, pretty much, all to it.

One fundamental thing to understand when designing models and working with MongoDB is that when you *retrieve* data from a Collection, the database engine works to locate complete *Documents* vs. individual pieces of JSON-formatted data stored somewhere inside those Documents. 

Say, your Sales Order database model looks like a JSON document with an embedded array of Order Items:

```json
{ 
  "orderNumber": 123,
  "delverToUser": 456,
  "billToUser": 789,
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

Sales Order Documents are stored in the Sales Order Collection in your MongoDB Database, and after some time, there will be a bunch of Documents in it (assuming that your selling business is going well). 

Now, let's say you want to see total sales for Item with sku=2001. *Hypothetically*, you query the Sales Order Collection to return `qty` where `"sku": 2001`, so you naturally expect to get back pieces like this:

```
  {"sku": 2001,
   "qty": 10}
```

No. What you get back is a set of complete Sales Order Documents that contain `"sku": 2001`. And then you need to do more work to pull required pieces from the dataset returned.

This is the biggest difference between Relational and MongoDB NoSQL philosophies. In Relational, each data component anywhere across your database is equally easy accessible - a simple query away. In MongoDB, you can retrieve values from fields at the top layer of the Document structure in a single query, but getting to data inside embedded objects takes extra steps. Make sure you understand that you *can* (and should) index embedded objects so that the query searching for specific data in them works fast. The point is - that query is not going to return you the *embedded* data - it will return *Documents* that *contain* the embedded data you're looking for.

Let's continue with our example. We indexed our Sales Order Collection on "items.sku", and when we ran a query to get data for `"sku": 2001`, the engine, using the index, quickly located all Sales Orders with that Item in them. Next, we need to shake off any extra data and build a small subset to be returned back from the database to the querying app over the network wire. 

So, for each Sales Order Document retrieved, we need the engine to perform a set of additional operations. First, we need to *reduce* the array of Items to remove everything except `"sku": 2001`. Operations inside Documents cannot use indexes, as MongoDB indexes, by definition, link *Documents* to values in the index, not arbitrary parts of Documents to arbitrary values. So, the Items array reduce operation will be done as in-memory manipulation over the JSON object vs. as a search within an indexed structure. 

MongoDB has a rich set of methods to handle JSON object processing. One way to extract just the item we need is to *unwind* (flatten) the array of "items". That will make the document looking like this:

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

See, that we also carry over the Order-level information in the intermediate dataset, as, realistically, we would need not just the `qty`, but also some of the Order details.

Next, we filter the intermediate document to select only element(s) with "items.sku"=2001 and we get our final result:

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

Not too difficult.

What does this mean for the developer? Design your data model in such a way that most frequently needed information can be extracted in a single query, without having the engine digging inside embedded objects for it. Back to the example above, naturally, if you have just a few Item lines per Order, storing everything in a single Document makes sense. Especially, if your business model is focused on the Order level processing. However, if your business model is Item-focused and your Orders contain many Items - you should consider storing each Order Item Line as a Document in a separate OrderItem Collection. In the latter case, you may decide to copy some Order information that doesn't change into each OrderItem collection Document and/or use the `lookup` functionality to link your OrderItem and Order Collections when retrieving data from both together. Sounds like a Relational design? Yes. Don't think of NoSQL as "no relations". NoSQL is about breaking the artificial limitations of the Relational model to expand usability, but keeping the smartness of it separating data into manageable and easily navigatable related sets.

And of course, the ultimate goal is, as always in the App-Database design, to minimize the amount of data traveling from the DB into the App. No filtering in the App - always reduce in the database.
<br>
So, we got the high-level of MongoDB, next, let's look a little deeper at Schema design and implementation specifics