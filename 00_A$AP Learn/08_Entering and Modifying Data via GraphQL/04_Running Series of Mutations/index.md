### sendUserAndOrder Mutation

On the GraphQL client, the developer can combine several named queries *or* mutations into a single *request*.

Check out `doc/graphql-grahiql-samples.md` Mutations, Create User and Order:

```
mutation sendUserAndOrder($user_data_input: CreateUserInput!, $order_data_input: CreateOrderInput!) {
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
  createOrder(input: $order_data_input) {
    order_id
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

Here we have our two mutations put together, literally, one after the other

### Client-Server Terminology Confusion

As a matter of fact, above, we have *one* mutation - `sendUserAndOrder`. The mutation has *two fields* - `createUser` and `createOrder`. That is how the *client* sees this syntax, and, sure enough, the word `mutation` is used only once, at the top, wrapping together the two *mutations* that are defined in the server GraphQL schema.

The official [GraphQL documentation](https://graphql.org/learn/queries/) talks about Queries and Mutations from the front end point of view first. Then they talk about [Query and Mutation Schema Types](https://graphql.org/learn/schema/#the-query-and-mutation-types). Here's the clue: in the client-side terminology, Queries and Mutations are *Requests*, whereas for the backend developer - they are *Types*. And those are *not* the same things. 

### Query Fields are Processed in Parallel, Mutation Fields - in Sequence

What does this terminology twist mean for the backend developer? Nothing. Ignore it.

Though, get the *parallel* vs. *sequential* statement above as you may hear it presented as the *key difference between queries and mutations*.

There is definitely some potential usefulness in the fact that multiple mutations put into one request (fields of one mutation) are executed by the server sequentially. You would hope so, by looking at `createUser` put above `createOrder`: you'd want the User be available by the time its Order is being rolled in. However, GraphQL does not guarantee *that*. It only guarantees that the steps will be executed one after the other.

If you run `sendUserAndOrder` from GraphiQL, the 1st step will likely fail (if you already have this user created). The 2nd step will complete successfully.

From your REST development experience you know that each call must be executed separately, and the client must wait for an acknowledgement to arrive before sending downstream dependant calls. Let's suggest the same to GraphQL client developers and shake off this *parallel vs. sequential* stuff as not so much relevant in real practice.
<br>
This completes the development tasks. The very last step is to generate the client-side GraphQL schema extract!
