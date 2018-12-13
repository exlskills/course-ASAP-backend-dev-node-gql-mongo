### mutateAndGetPayload Finction

The GraphQL mutation schema is in `src/relay-mutations/user/create-user-mutation.js`. We looked at its components already in the "GraphQL Data API" chapter. After the query work, the mutation components are very recognizable. 

The schema defines the fields that come in (as part of the *HTTP request*) and the code that is executed when the mutation is launched from the client, as well as the content that is sent back (in the *HTTP reply*).

As we put debug logging into the schema code, we have a full blown function written in it:

```
mutateAndGetPayload: (inputFields, viewer, info) => {
    logger.debug(`in mutateAndGetPayload Create User `);
    logger.debug(`  inputFields ` + JSON.stringify(inputFields));
    logger.debug(`  viewer ` + JSON.stringify(viewer));
    logger.debug(`  info ` + JSON.stringify(info));
    return createUser(inputFields, viewer, info);
  }
```

As we already know, this can be minimized all the way down to

```
mutateAndGetPayload: createUser
```

### createUser Function

Similarly to coding a designated "buffer" resolver function for query processing, in mutations we use write a `mag` - "mutate and get" function that isolates the GraphQL process from calling database operations directly.

`src/relay-mutate-and-get/user-mag.js` runs the process logic and calls the data layer operations that run `mongoose` stuff.

`conditionsToCheck` reflect the data validation steps to ensure that neither `username` nor `primary_email` are duplicated:

```
    const conditionsToCheck = [
      {
        match: { username: user_data.username },
        msg_id: 'DU01',
        msg: 'Username already exist'
      },
      {
        match: { primary_email: user_data.primary_email },
        msg_id: 'DU02',
        msg: 'Primary email found in another existing profile'
      }
    ];
```

We could have built *unique* indexes in MongoDB on those fields to enforce insert/update rejection at the database level if duplicates are passed in, but checking the conditions explicitly enables better error reporting. `msg` and `msg_id` are used here to control messaging being sent back to the client. Limiting just to `msg` doesn't work well for *internationalization*. The concept of adding `msg_id` allows overriding the `msg` in the language of the client's user locale (not in scope of the demo).

The `match` properties are conveniently passed directly into the `basicFind` function, although, this design brings the code too close to being `mongoose`-specific - compromising the *buffering* design.

If the validation passes, `UserCud.createUser` is called to *create* the User Document in MongoDB.

The return values are the User ID if the Document gets create (converted to the Global ID) and the uniform `completionObj` that details the process outcome.


Next, we'll look at the MongoDB operation that *creates* the Document in the User Collection
