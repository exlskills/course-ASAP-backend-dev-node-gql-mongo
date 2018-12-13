### createUser MongoDB Call

`src/db-handlers/user/user-cud.js` contains the `mongoose` code inserting the Document: 

```
import User from '../../db-models/user-model';
import { id_gen } from '../../utils/url-id-generator';
import { logger } from '../../utils/logger';

export const createUser = async user_data => {
  logger.debug(`in createUser`);
  const user_id = id_gen();
  const userObject = { _id: user_id, ...user_data };
  try {
    await User.create(userObject);
    return user_id;
  } catch (err) {
    return Promise.reject('Error creating User', err);
  }
};
```

Notice, we generate the `_id` directly in the code so that we can easily return the ID back to the client. Alternatively, we may let `mongoose` creating the `_id` as defined in the User Schema, but then there is no immediate way to extract it other than by querying the Collection. Arguably, such a follow-up query, using one of the unique keys, e.g., `username`, would serve as the ultimate confirmation that the Document has been added, but sounds rather excessive in a normal *insert* flow. The database layer should be reliable enough to ensure that the operation succeeded if no errors have been returned while executing it.

In the User Order creation flow, we check that the generated `_id` doesn't already exist in the Collection before passing it in. This has been discussed in the "Mongo DB Datastore" chapter. 

The error is returned as `Promise.reject()` vs. `throw new Error()` we used in queries. The end effect is the same, so it is more of a coding style/standard decision which approach to use. However, in the mutation process design, we catch all errors in the "Mutate-and-Get" function (`src/relay-mutate-and-get/user-mag.js`), e.g., 

```
  catch (error) {
    return {
      completionObj: {
        code: '1',
        msg_id: 'EU01',
        msg: error.message,
        processed: 0,
        modified: 0
      }
    };
  }
```

Instead of an passing back an "error" like with queries, we always return the `completionObj`, and the client should look for the process result in there. Logically, query errors are *exceptions*, whereas mutation errors can be due to the quality of data or business process flow gaps, so they should be reported in a more casual way. Controversially, GraphQL does not support setting HTTP response error codes like REST does. Some view it as a limitation, and it is for traditionalists who used to checking that vs. the body of the response.

Next, we'll look at a more substantial insert operation - createOrder