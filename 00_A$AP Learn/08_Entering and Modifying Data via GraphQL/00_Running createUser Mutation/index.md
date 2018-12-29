### createUser Mutation

Let's get this out of the way: *mutation* sounds like an awful term to use in the everyday IT practice, unless your software application field is biology or gaming. Anything - *modifications*, *updates*, *inserts* - is completely normal to a developer's ear. But as the industry evolves (or *mutates*) - we've got to get used to new terms. As long as they don't convert IT developers into *mutants* - everything is completely fine. Moving on. 

As we did when working with the queries, with the GraphQL server running in the `node-dev` container's shell, open a browser on the host and navigate to the GraphiQL tester `http://localhost:8080/graph`. This time, though, let's expand the `QUERY VARIABLES` section at the bottom of the left pane and copy this content from `doc/graphql-grahiql-samples.md` ( Mutations, Create User, Variables ) into there:

```json
{
  "user_data_input": {
    "user_data": {
      "full_name": "Super User",
      "username": "s_user",
      "primary_email": "super@example.com",
      "primary_locale": "en"
    }
  }
}
```

Then copy the mutation's content into the top part of the left pane (you can leave the queries in there if you already have things typed in):

```
mutation sendUser($user_data_input: CreateUserInput!) {
  createUser(input: $user_data_input) {
    user_id
    completionObj {
      code
      msg_id
      msg
      processed
      modified
    }
  }
}
```

Notice `$user_data_input` in two places in the mutation code - matching the name defined in the Query Variables section. In GraphiQL, variables can be used in queries as well - rather than writing long conditions directly inside `()`. For mutations, as we have to explicitly provide the input type (`CreateUserInput!`) - use of variables is the common practice. The `!` at the end of the Input Type's name reflects everyone's excitement about this technology. 

### Running the Mutation

As this is going to *mutate* the data in our MongoDB, we need to be prepared to *un-mutate* if we want to test certain transactions again. Conveniently, MongoDB Compass has a quick *delete* option at the Document level in the Collections GUI, so have Compass running.

The execution on the GraphiQL side is the same as when running queries - the run-like button. You should get this result in the right pane:

```
{
  "data": {
    "createUser": {
      "user_id": "<some id>",
      "completionObj": {
        "code": "0",
        "msg_id": "001",
        "msg": "User created",
        "processed": 1,
        "modified": 0
      }
    }
  }
}
```

In Compass, you can see a new User Document added with the ID returned by the mutation process.
<br>
Next, let's review the code that carries out the flow