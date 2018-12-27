### Demo Project db-models Features

DB models are stored in the `src/db-models` code folder of the demo project.

Each model imports `mongoose`:

```
import mongoose from 'mongoose';
```

## URL-ready IDs

User, Item and Order models illustrate the use of custom-generated Document `_id`:

```
import { id_gen } from '../utils/url-id-generator';
...
    _id: {
      type: String,
      default: id_gen
    }
```

Custom function `id_gen` (see `src/utils/url-id-generator.js`) creates a 12-pos unique ID that can be used as part of a public URL identifying the corresponding business object - a *SEO-friendly URL* that would contain some verbal tag-like definition of the object, followed by this unique ID. SEO-friendly URLs help improve visibility of the site to Internet search engines. In the scope of the Use Case, *Item* is a definite candidate to be identified via a SEO-friendly URL. URL links for Users and Orders may also look aesthetically more pleasing for website customers if built using verbal identifiers vs. series of random-looking characters.

As the URL must be unique for each object on the server, the `_id` is used as the last part of the URL to guarantee the uniqueness. The ID must be long enough to accommodate the projected size of the dataset, URL-ready (no special characters), and, lastly, the URL should be "profanity-proof": free of randomly generated letter sequences that match undesirable words. That is what `id_gen` is for. The 12-pos size provides a very good randomness for less than 1 Million IDs and Ok for much larger sets as well. When generating an ID for a particular type of Document, you may:

- let `mongoose` call the function as per `default: id_gen` and hope that the generated ID is not already in the Collection. If it is, the Create operation will fail with a MongoDB-generated error message looking like
  ```
  MongoError: E11000 duplicate key error collection: web_dev.user_order index: _id_ dup key: { : "1UtNnZBYUHDx" }
  ```
- generate the ID in the code and query the Collection to ensure that it is not in there, re-generate if found
- catch and analyze the MongoDB error to check if the Create operation failed specifically due to the ID duplication error, re-generate the ID and retry

When Documents in the Collection are created infrequently, the in-code ID generation and validation method absolutely guarantees that the Create operation succeeds. Even when there several Documents being created at the same moment, the probability that `id_gen` suggests the same ID for more than one of those is practically negligible. The error checking method is, obviously, bullet-proof, but requires extra coding and relying on the error format to be consistent. We'll see both methods used in the demo project code when we review mutations in the "Entering and Modifying Data via GraphQL" chapter.
 

## Automatic Last Modified Timestamps

For objects (either Documents or embedded) that we'd like to track modification timestamps on, the following is added to the Schema:

```
  {
    timestamps: { createdAt: 'created_at', updatedAt: 'updated_at' }
  }
```

This signals `mongoose` to generate the timestamps automatically and place into the fields according to our naming convention (`created_at` and `updated_at`), before passing data to MongoDB. 

As a note, not all `mongoose` operations manage timestamps on *embedded* objects. E.g., `save` operation updates the embedded objects' timestamps but `Model.update`. Check the latest `mongoose` documentation for details when coding with this feature. When the operation does manage timestamps, `mongoose` will perform timestamp updates as required up the embedding chain of objects when a lower-level object is modified.


## Index Creation Via the Schema

Note the `index: true` property in the demo project Schemas on fields that would likely be used in queries and filters, e.g., in `src/db-models/user-model.js`:

```
    username: {
      type: String,
      index: true,
      unique: true
    },
```

At startup, `mongoose` validates that all indexes listed in the Schema are present in the database and generates those that are not, using the background index generation mode by default. Note that `mongoose` will *not* modify or delete existing indexes, so this feature is a safe and convenient way setting up dev instances as well as deploying new indexes to production. Unless your database is large and strained, background index generation should not cause a noticeable slowdown. 

`mongoose` enables creating additional indexes via the `index` command on the Schema object, e.g., in `src/db-models/user-order-model.js`:

```
UserOrderSchema.index({ user_id: 1, payer_id: 1 });
UserOrderSchema.index({ user_id: 1, order_date: -1 });
```

This method can be used to create Compound and Text indexes. Per MongoDB limits, a Collection can have up to 64 indexes in total, also just a single Text index.


## Schema "Unique" Property

In the `username` definition example above, `unique: true` is set. This property applies to the *index*. In the version of `mongoose` used in the demo project, it appears that this property must be set *before* the index is first created, otherwise, `mongoose` will not *recreate* the index. If the non-unique index already exists, update it via Compass or delete and let `mongoose` create it.

A confusing part about this property is that `mongoose` itself does not *pre-validate* the uniqueness at the Model level, so the database operation gets invoked and errored out by the MongoDB engine. Most other Schema-level validations are done by the `mongoose` code, reported differently and block the database call execution if failed. 

`mongoose` internal Schema-level validations and error reporting are not in scope of this course and not used in the demo project


Let's review each of the Models in scope next
