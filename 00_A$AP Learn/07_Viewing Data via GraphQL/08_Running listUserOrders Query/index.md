### listUserOrders Query

With the GraphQL server running in `node-dev` container's shell, open a browser on the host and navigate to the GraphiQL tester `http://localhost:8080/graph`. Copy this query content (from `doc/graphql-grahiql-samples.md`, Queries, List User Orders) into the left pane of GraphiQL window (replacing the commented out instructions, if you haven't cleared them out before; although, you can leave the `listUsers` query in the pane):

```
query listUserOrders {
  listUserOrders(user_id: "VXNlcjoxRTRZbzExWTNyOWE=", first: 3, orderBy: [{field: "order_date", direction: ASC}]) {
    edges {
      node {
        id
        payer_id
        order_date
        order_items (first: 10) {
          edges {
            node {
              item_id
              desc
              quantity
              amount
              item_category
              item_details
            }
          }
        }
      }
    }
  }
}
```

Hit the "Execute Query" run-like looking button on the top bar, next to the GraphiQL logo, and select `listUserOrders` if you have more than one query in the pane. You should get this in the right (result) pane:

```
{
  "data": {
    "listUserOrders": {
      "edges": [
        {
          "node": {
            "id": "VXNlck9yZGVyOjIwTk1mcDNNbExsRA==",
            "payer_id": "VXNlcjoxRTRZbzExWTNyOWE=",
            "order_date": "2018-01-01T08:00:00.000Z",
            "order_items": {
              "edges": [
                {
                  "node": {
                    "item_id": "SXRlbToxSEVPeDZGbkM3Y00=",
                    "desc": "Sample Food Item",
                    "quantity": 5,
                    "amount": 10.13,
                    "item_category": "Food",
                    "item_details": "{\"shelf_life_mo\":3,\"weight_lb\":0.41,\"packaging\":\"frozen-pack\",\"special_handling\":{\"before_shipping\":\"freeze\",\"in_transit\":\"refrigerate\"}}"
                  }
                },
                {
                  "node": {
                    "item_id": "SXRlbToxR1ZsMjFscTBTOHA=",
                    "desc": "Sample Consumer Item",
                    "quantity": 2,
                    "amount": 15.99,
                    "item_category": "Consumer",
                    "item_details": "{\"weight_lb\":1.3,\"color\":\"black\"}"
                  }
                }
              ]
            }
          }
        },
        {
          "node": {
            "id": "VXNlck9yZGVyOjIwTk1mYWFBdFNITA==",
            "payer_id": "VXNlcjoxRTRZbzExWTNyOWE=",
            "order_date": "2018-02-01T08:00:00.000Z",
            "order_items": {
              "edges": [
                {
                  "node": {
                    "item_id": "SXRlbToxSEVPeDZGbkM3Y00=",
                    "desc": "Sample Food Item",
                    "quantity": 10,
                    "amount": 20.26,
                    "item_category": "Food",
                    "item_details": "{\"shelf_life_mo\":3,\"weight_lb\":0.41,\"packaging\":\"vacuum\"}"
                  }
                },
                {
                  "node": {
                    "item_id": "SXRlbToxR1ZsMjFscTBTOHA=",
                    "desc": "Sample Consumer Item",
                    "quantity": 3,
                    "amount": 22.8,
                    "item_category": "Consumer",
                    "item_details": "{\"weight_lb\":1.3,\"color\":\"yellow\"}"
                  }
                }
              ]
            }
          }
        }
      ]
    }
  }
}
```

Everything looks familiar from the what we've already seen working with the `listUsers` query. Notice `item_details` represented as a JSON *escaped* (filled with `\` as needed to preserve double quotes) string. This is by virtue of defining `item_details` as a GraphQLString in the Object Type. Unlike MongoDB, GraphQL requires object structures to be defined in order to display them as response fields. The closest we can get to a flexible structure output - by using `GraphQLUnionType` that enables a field to take one of the several pre-defined Object-like structures. However, in modern software a well formatted JSON is easily converted to an object, so the client should be able to work with this string just fine.


Next, let's take a look at the underlying code implementation for listUserOrders.