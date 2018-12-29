### mutateAndGetPayload Finction

The GraphQL mutation schema is in `src/relay-mutations/user/create-user-mutation.js`. We looked at its components already in the "GraphQL Data API" chapter. After the query work, the mutation components are very recognizable. 

The schema defines the fields that come in from the client (as part of the *HTTP request*) and the code that is executed when the request comes in to the server, as well as the content that is sent back to the client (in the *HTTP reply*).

As we put debug logging into the GraphQL schema code, we have what looks like a full blown function written in there:

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

Similarly to coding a designated "buffer" resolver function for query processing, in mutations we write a `mag` - "mutate and get" function that isolates the GraphQL process from the database operations.

`src/relay-mutate-and-get/user-mag.js` runs the process logic and calls the data layer function that executes `mongoose` stuff.

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

We have built *unique* indexes in MongoDB on those fields to enforce insert/update duplicates rejection at the database level, but, arguably, checking the conditions explicitly enables better error reporting than analyzing MongoDB error messages. In the "Demo Project Mongoose Models and Schemas" lesson, it was pointed out that `mongoose` doesn't validate uniqueness at the Schema level. Obviously, validating in the database provides the ultimate guarantee, whereas the programmatic validation as the above is only *suggestive* - as there is no transaction processing around the flow. Extra querying is a slight performance hit as well. So, choose a proper design depending on the use case. Later, in "Running createOrder Mutation" we'll briefly look at how MongoDB error analysis can be coded and used.

`msg` and `msg_id` are used here to control messages sent back to the client. Limiting just to `msg` doesn't work well for *internationalization* (or *i18n* as the abbreviation often used in IT). The concept of adding `msg_id` allows overriding the `msg` per the client's user locale (not in scope of the demo project).

The `match` properties are conveniently passed directly into the `basicFind` function, although, this design brings the code too close to being MongoDB-specific - compromising the data layer *buffering* design.

If the validation passes, `UserCud.createUser` is called to *create* the User Document in MongoDB.

The return values are the User ID if the Document gets created and the uniform `completionObj` that details the outcome of the process. The ID is sent back converted to the Global ID - keeping the design concept that all IDs passing via GraphQL should be in that special format.
<br>
Next, we'll look at the MongoDB operation that creates the Document in the User Collection
