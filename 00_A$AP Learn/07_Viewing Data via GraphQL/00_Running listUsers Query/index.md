### listUsers Query

With the GraphQL server running in `node-dev` container's shell (`npm start`), open a browser on the host and navigate to the GraphiQL tester `http://localhost:8080/graph`. Copy this query content (`doc/graphql-grahiql-samples.md`, Queries, List Users) into the left pane of the GraphiQL window (get rid of the comments with the GraphiQL instructions, if you haven't cleared them out before):

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

Here we have the query *arguments* listed in the parentheses, followed by the list of fields that we want to get back (starting with `edges`), as well as the `pageInfo` - clearly has something to do with *pagination*

Hit the "Execute Query" run-like looking button in the top bar, next to the GraphiQL logo. You should get this in the right (result) pane:

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

In the `node-dev` shell, you can see some debug-level log statements printed. It is definitely cooking.


Everything looks pretty simple and logical. Now, let's review the exact syntax and flow to ensure there is no mystery left in what is done and how.
