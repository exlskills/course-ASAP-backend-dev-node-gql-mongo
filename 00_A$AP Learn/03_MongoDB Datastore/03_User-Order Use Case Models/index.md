### Use Case db-models

DB models are stored in `src/db-models` code folder. 

Each model imports `mongoose`:
```
import mongoose from 'mongoose';
```

## URL-friendly IDs

User, Item and Order models illustrate the use of a custom unique identifier for the Collection `_id`:
```
import { id_gen } from '../utils/url-id-generator';

...

    _id: {
      type: String,
      default: id_gen
    }
```

The custom function `id_gen` creates a 12-pos unique ID that can be used as part of a public URL when linking to a particular object from the client browser. So called SEO Friendly URLs are those that contain a clear verbal tag-like definition of the object for better exposure to internet search engines, yet the URLs must be unique for the given server. Using a designated custom `_id` helps generating the URL once for the Document, then use it as part of the easy access to the Document by its ID. The ID must be long enough to be unique, URL-ready (no control symbols), and, lastly, the URL should be "profanity-proof": free of randomly generated letter sequences that match undesirable words. This is what `id_gen` is for. The 12-pos size provides a good randomness for less than 1 Million IDs, regardless, when generating the ID, it should be checked for uniqueness by querying existing Documents.

## Automatic Last Modified Timestamps

For objects (either Documents or embedded) that we'd like to track the Last Modified timestamps on, the following is added to the Schema:
```
  {
    timestamps: { createdAt: 'created_at', updatedAt: 'updated_at' }
  }
```
This signals `mongoose` to generate the timestamps automatically and place into the fields according to our naming convention, before passing the data to the MongoDB. Note, that the update timestamp on embedded objects is only managed by `mongoose` when using the `save` function on the Document object. However, check the latest `mongoose` documentation if the functionality has been expanded to other methods, such as `Model.update` when you need to use it.
Generally, `mongoose` will trace the timestamp updates as required up the embedding chain of objects when a lower-level object is modified.

## Index Creation Via the Schema

Note the `index: true` property on the fields that would likely be used in queries and filters. At startup, `mongoose` validates that all indexes listed in the Schema are present in the database and will initiate a background index generation for those that are not. `mongoose` will not modify or delete existing indexes, which makes this feature convenient when setting up new dev instances with the database as well as deploying code changes to production, with new fields or indexes added to the Schema. Unless your database is large and strained, background index creation in Production should not cause a noticeable slowdown. 

Per MongoDB limits, a Collection can have up to 64 indexes. `mongoose`, however, allows only one index creation statement per Schema (in addition to the field-level `index` property that corresponds to a single-field index over that field). This severely limits index defining capabilities of `mongoose` Schemas. Additional steps must be taken to create all the necessary Compound or Text indexes in the Collection. In dev environments, the presence of Compound indexes is not required other than for performance testing. In production, indexes can be added manually via the Compass. 

## User Model

The Schema contains several basic demo fields, one of which illustrates marking a field as `required` and supplying a default value to it
```
    primary_locale: {
      type: String,
      default: 'en',
      required: true
    }
```
Technically, if the default is specified, `mongoose` will add the field with that value in it, so including both properties is redundant, unless we want to stress the importance of the field and (hypothetically) cover update-time validations by `mongoose` on Documents inserted by external systems.

The Model statement links the Schema to the Collection named `user`:
```
export default mongoose.model('User', UserSchema, 'user');
```

## Item Model

The Schema illustrates the use of a free-form embedded object:
```
    item_details: {
      type: Object,
      default: {}
    },
```
This indicates that any multi-property object can be placed into the `item_details` field of the Item document, and if one is not provided - `mongoose` will create an empty document placeholder in that field

A more controlled way of Object embedding is seen here:
```
    item_price: {
      type: ItemPrice,
      required: true
    }
```
`ItemPrice` is a Schema defined in another file (`src/db-models/item-price-model.js`) and imported at the top of the `src/db-models/item-price-model.js`

# Item Price Model (Schema only)

Note that `src/db-models/item-price-model.js` `export default` statement exports a Schema vs. a Model as  `ItemPrice` is an object designed to be embedded into another Document - in this case, into the Item Document in the `item` Collection.

`_id: false` signals `mongoose` to skip ID generation for the embedded ItemPrice object, however, the modification timestamps on the ItemPrice object will be present and maintained.

## User Order Model

In our design, Orders and Order Items are stored in one Collection called `user_order`. 

Each Document contains IDs of the User (`user_id`) and Payer (`payer_id`), which point to Documents in the User Model. The `ref: 'User'` field in the schema enables the use of the `mongoose`-proprietary lookup engine [Populate](https://mongoosejs.com/docs/populate.html). `mongoose` Populate is not utilized in the demo as we're limiting dependencies on proprietary features, and the `ref` property is included for information only.

Both `user_id` and `payer_id` have single field indexes created, which assist finding all Orders by the given User or paid for by the User-Payer. 

Order Date will be maintained transparently as we specify the `Date` type in the Schema
```
    order_date: {
      type: Date
    },
```

The Order Items is an embedded array of Objects defined the `OrderItemSchema`, imported from `src/db-models/order-item-model.js`:
```
    order_items: {
      type: [OrderItemSchema]
    }
```

# Order Item Model (Schema)

Similarly to the Item Price, Order Item is a Schema-only in our design, with a corresponding Model (Collection).

A designated unique `_id` is added to each embedded Order Item object, the default MongoDB `ObjectId` type is used:
```
    _id: {
      type: mongoose.Schema.Types.ObjectId,
      auto: true,
      index: true
    },
```
The `auto` flag ensures that the `_id` is generated. By default, MongoDB does not generate `_id` on embedded objects. From the business solution standpoint, we can benefit from having a unique identifier at the Order Item level for tracking, and MongoDB ObjectID looks like an easy way to assign one. Also note the `index: true` property on this field - again, MongoDB would not auto-index `_id` on an embedded field. Arguably, we might have used a different field name all together, but `_id` looks consistent with how objects are marked in MongoDB.

Note the `index: true` for the `item_id` field and the `type: Object` designation for the `item_details`. We'll see later how the built-in object handling capabilities of JS allow us "merging" the `item_details` from the Item Document with those specified for the Order Item, overriding matching properties with the more specific values entered on the Order.

That's all with MongoDB for now, lets move on to see how the NodeJS App server runs!