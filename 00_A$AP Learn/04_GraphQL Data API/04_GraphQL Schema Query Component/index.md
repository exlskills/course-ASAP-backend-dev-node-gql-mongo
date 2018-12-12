### Sample GraphQL Query Schema

Let's look at `src/relay-queries/user/user-order-list-query.js`

```
import { GraphQLNonNull, GraphQLID } from 'graphql';
import { connectionArgs } from 'graphql-relay';
import * as inputTypes from '../input-types-get-query';
import { UserOrderConnection } from '../../relay-models/user-order-type';
import { resolveListUserOrders } from '../../relay-resolvers/user-order-resolver';

export const listUserOrders = {
  type: UserOrderConnection,
  description: 'User Orders',
  args: {
    orderBy: {
      type: inputTypes.OrderByType
    },
    filterValues: {
      type: inputTypes.FilterValuesType
    },
    resolverArgs: {
      type: inputTypes.QueryResolverArgsType
    },
    user_id: {
      type: new GraphQLNonNull(GraphQLID)
    },
    ...connectionArgs
  },
  resolve: (obj, args, viewer, info) =>
    resolveListUserOrders(obj, args, viewer, info)
};
```

- `type: UserOrderConnection` defines the *output* Object Type - it is a *Connection* defined in `src/relay-models/user-order-type.js`. As we noted briefly in the previous lesson, Connections are used for paging, when we bring "one page at a time". We'll review Connections work in more details when we talk about serving data
- `args` - those are the *input* fields the client should be passing with the query. Those of type `GraphQLNonNull` are required, others are optional
  * `orderBy`, `filterValues` and `resolverArgs` are *custom* *Input* types that we are defining in `src/relay-queries/input-types-get-query.js` and use throughout the project. Those are the inputs into our custom paging engine that we'll review
  * `...connectionArgs` - in JS, `...` followed by the name of the object is called *spread*. It evaluates to the list of all components of the object. `connectionArgs` comes from `graphql-relay` and controls paging
- `resolve` defines the query *resolver* function: `(obj, args, viewer, info) => resolveListUserOrders(obj, args, viewer, info)` with these parameters (we'll review them in more details later)
  * `obj` - this is the *source* object, similar to the one used in the Object Type processing. In the context of a query *resolver* (vs. Object Type's *field* resolver), the *source* would only be available if the query is "embedded" into a parent query process - then the *source* here is the return object of the parent query resolver. Otherwise, the `obj` here is a `null`. You will rarely see `obj` used in query resolvers, but we keep the four-parameter signature common in all calls across the project for consistency
  * `args` - the values coming from the client, per the schema definition above
  * `viewer` - the object defining the *user* submitting the query, e.g., the user logged in into the client browser 
  * `info` - a work object supplied by GraphQL containing request operation information in a raw JSON form. The parameter is not used in the processing and can be omitted

  Note, that in the demo we always explicitly define the for parameters as the arguments to the *arrow function* call. This *short and implicit* notation also works:
  ```
  resolve: resolveListUserOrders
  ```

  We can omit the list of parameters and the arrow function sign `=>`, just provide the name of the function for the GraphQL engine to call - and it will call it passing the four parameters. Pick whichever notation you prefer, just keep in mind that the short notation assumes that there is nothing else after `resolve:` other than the function name. The arrow function notation lets you put some more code directly into the schema, e.g., 

  ```
  resolve: (obj, args, viewer, info) => {
   // This is some comment
   logger.debug(` we got this far`);
   return resolveListUserOrders(obj, args, viewer, info);
  }
  ```

  Once you put `{}` into the arrow function - it becomes a multi-line actual function containing a `return` statement. Without `{}`, you have to limit the writing to the function name with the list of parameters.


Lastly, let's take a quick look at how mutations are defined in the GraphQL schema, before jumping into the code and running real flows