### ListUsers Query

With the GraphQL server running in `node-dev` container's shell, open a browser and the host and navigate to the GraphiQL tester `http://localhost:8080/graph`. Copy this query content (from `doc/graphql-grahiql-samples.md, Queries, List Users) into the left pane of GraphiQL window (replacing the commented out instructions, if you haven't cleared them out before):

```
query listUsers {
  listUsers(first: 3, orderBy: [{field:"full_name",direction:ASC}]) {
    edges {
      node {
        id
        full_name
        username
        primary_email
        primary_locale
      }
      cursor
    }
    pageInfo {
      hasNextPage
      hasPreviousPage
      startCursor
      endCursor
    }
  }
}
```

Here we have the query *arguments* listed in the parentheses, followed by the list of fields that we want to get back (starting with `edges`), as well as the `pageInfo` that clearly has something to do with *pagination*


Hit the "Execute Query" run-like looking button on the top bar, next to the GraphiQL logo. You should get this in the right (result) pane:
```
{
  "data": {
    "listUsers": {
      "edges": [
        {
          "node": {
            "id": "VXNlcjoxRTRZbzExWTNyOWE=",
            "full_name": "Jane Doe",
            "username": "jane01",
            "primary_email": "jd@example.com",
            "primary_locale": "en"
          },
          "cursor": "bW9uZ286MUU0WW8xMVkzcjlhfDB8MA=="
        },
        {
          "node": {
            "id": "VXNlcjoxRWc2ZDFhRkw4czc=",
            "full_name": "John Public",
            "username": "john01",
            "primary_email": "jp@example.com",
            "primary_locale": "en"
          },
          "cursor": "bW9uZ286MUVnNmQxYUZMOHM3fDB8MQ=="
        }
      ],
      "pageInfo": {
        "hasNextPage": false,
        "hasPreviousPage": false,
        "startCursor": "bW9uZ286MUU0WW8xMVkzcjlhfDB8MA==",
        "endCursor": "bW9uZ286MUVnNmQxYUZMOHM3fDB8MQ=="
      }
    }
  }
}
```

In the `node-dev` shell, you can see some debug-level log statements printed.

This looks very simple, but let's review the exact syntax and flow to ensure there is no mystery left in how this is done, so you can use this base sample to right your own code
