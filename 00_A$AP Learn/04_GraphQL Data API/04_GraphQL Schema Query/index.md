### GraphQL Query Schema

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

Here's what we've got:

- `type: UserOrderConnection` defines the *output* Object Type - it is a *Connection* defined in `src/relay-models/user-order-type.js`. As we noted briefly in the previous lesson, Connections are used for *paging*, when the client receives "one page at a time" and has to explicitly request each page in a subsequent query, adjusting the query arguments. We'll review how Connections work in more details when we talk about serving data
- `args` - those are the query *input* fields the client should be passing with the query. Fields that have `GraphQLNonNull` in their definition are required, others are optional
  * `orderBy`, `filterValues` and `resolverArgs` are *custom* *Input* types that we define in `src/relay-queries/input-types-get-query.js` and use throughout the demo project. These arguments are used in our implementation of the Relay paging engine that we'll review in detail
  * `...connectionArgs` - in JS, `...` followed by the name of the object is called *spread*. It evaluates to the list of all components in the object. Object `connectionArgs` comes from `graphql-relay` and controls Relay-defined paging
- `resolve` defines the query *resolver* function: `(obj, args, viewer, info) => resolveListUserOrders(obj, args, viewer, info)`. Briefly for now, the four arguments are:
  * `obj` - this is the *source* object, *similar* but *NOT* the same as the one used in the Object Type processing that we saw in the previous lesson. In the context of a *query resolver* (vs. *field resolver* in the Object Type), the *source* would only exist if the query is "embedded" into a parent query process. In that case the *source* here is the return object of the parent query's resolver. Otherwise, the `obj` here is a `null` - just a placeholder, basically
  * `args` - the values coming from the client, the one listed in the schema definition right above
  * `viewer` - the object defining the *user* submitting the query, e.g., the user logged in into the client browser. More about this later
  * `info` - a work object supplied by GraphQL containing request operation information in a raw JSON form. The argument is generally not used in the processing and can be omitted. We'll review a use case where `info` can be utilized at the end of this chapter

  Note that in the demo we always explicitly define the for arguments as the arguments to the *arrow function* call. This *short and implicit* notation also works:

  ```
  resolve: resolveListUserOrders
  ```

  We can omit the list of arguments and the arrow function sign `=>`, just provide the name of the function for the GraphQL engine to call - and it will call it passing the four arguments. Pick whichever notation you prefer, just keep in mind that the short notation assumes that there is nothing else after `resolve` other than the function name. The arrow function notation lets you put some more code directly into the schema, e.g., 

  ```
  resolve: (obj, args, viewer, info) => {
   // This is some comment
   logger.debug(` we got this far`);
   return resolveListUserOrders(obj, args, viewer, info);
  }
  ```

  Once you put `{}` into the arrow function - it becomes a multi-line function that must use an explicit `return` statement. Sometimes it feels like JS shoots itself in the foot allowing all these different variations of the syntax, seemingly, just to save a few characters in the code. Yes, but less characters equate to faster code load into the browser. Whatever the reason is - JS syntax is what we have to live with in this line of work.


Lastly, let's take a quick look at how mutations are defined in the GraphQL schema, before jumping into the code and running real flows. Very close!