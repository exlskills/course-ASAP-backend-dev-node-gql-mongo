### From Schema to Query Syntax

As we saw in "Use Case and Project Components" chapter, in the demo project, the code folder structure related to query and mutation processing is rather expansive. In the design used, there are layers of exporting and importing that separate `src/schema.js` from the files where components are defined. There are quite a few `index.js` files to sort through. If you don't like this approach you don't have to use it. The opposite alternative is to code things right into `src/schema.js` as you can see in some tutorials. Good luck with that if you have more than a couple objects in scope. In a real app, the layered design is probably best to manage the complexity.

So, let's roll from the `src/schema.js` following the trail:

```
import queryMap from './relay-queries';
import mutationMap from './relay-mutations';
```

`src/relay-queries/index.js` contains the `queryMap`:

```
import user from './user';
import item from './item';

export default {
  getItem: item.getItem,
  getUser: user.getUser,
  listUsers: user.listUsers,
  listUserOrders: user.listUserOrders
};
```

In the *map*, the public query name to be used in the client, e.g., `listUsers`, is linked to its internal definition and implementation schema, e.g., `user.listUsers`.

Following down the path, `index.js` under `src/relay-queries/user` imports and exports individual query schemas. If you think this layer is excessive, you can import those query schemas directly into `src/relay-queries/index.js` - a matter of coding style preferences. 

Finally, the query definition code that we'll review below is in `src/relay-queries/user/user-list-query.js`

### Query Arguments

The arguments that the client can or must pass with the query are listed in the query schema `args` property:

```
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
    ...connectionArgs
  }
```

Per `graphql-relay` (`node_modules/graphql-relay/lib/connection/connection.js`; remember, we talked about ownership of `node_modules` on Linux), `connectionArgs` are as follows:

- after
- first
- before
- last

From VSC, you can get into `connection.js` by clicking on `connectionArgs` in the import statement. There is some interesting stuff in the file, e.g.

```
var connectionType = new _graphql.GraphQLObjectType({
    name: name + 'Connection',
    description: 'A connection to a list of items.',
    fields: function fields() {
      return _extends({
        pageInfo: {
          type: new _graphql.GraphQLNonNull(pageInfoType),
          description: 'Information to aid in pagination.'
        },
        edges: {
          type: new _graphql.GraphQLList(edgeType),
          description: 'A list of edges.'
        }
      }, resolveMaybeThunk(connectionFields));
    }
  });
```

and 

```
var edgeType = new _graphql.GraphQLObjectType({
    name: name + 'Edge',
    description: 'An edge in a connection.',
    fields: function fields() {
      return _extends({
        node: {
          type: nodeType,
          resolve: resolveNode,
          description: 'The item at the end of the edge'
        },
        cursor: {
          type: new _graphql.GraphQLNonNull(_graphql.GraphQLString),
          resolve: resolveCursor,
          description: 'A cursor for use in pagination'
        }
      }, resolveMaybeThunk(edgeFields));
    }
  }); 
```

Lots of words are in here that we've seen in the schema, in the query we put in, and in the result that came back:

- connection
- edge
- node
- cursor
- pageInfo

<br>
Perfect time to get all of this straight, so we can continue on to the more interesting stuff. 

