### From Schema to Query Syntax

As we mentioned earlier, the folder structure used for queries and mutations is rather expansive. It demonstrate the approach of exporting and importing layers of code, via `index.js` files, till we finally get to `src/schema.js`, where this code pulls everything in:
```
import queryMap from './relay-queries';
import mutationMap from './relay-mutations';
```

`src/relay-queries/index.js` contains the queryMap:
```
import user from './user';

export default {
  listUsers: user.listUsers,
  listUserOrders: user.listUserOrders
};
```
In the map, the public query name (`listUsers`, as used in the browser) is linked to its internal definition and implementation schema.

There is another `index.js` file under `src/relay-queries/user` which imports and exports individual schemas. If you think this layered import-export design is excessive - you can import the query schemas directly where you need them, this just a matter of coding style preferences. The code we're reviewing below is in `src/relay-queries/user/user-list-query.js`

## Query Arguments

The arguments that should be passed to the query are listed in the query schema `args` property:
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

Per 'graphql-relay' (`node_modules/graphql-relay/lib/connection/connection.js`), `connectionArgs` are as follows:
- after
- first
- before
- last

You can get into `connection.js` by clicking on `connectionArgs` in the import statement. You can see lots of interesting stuff in the file, e.g.
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


Perfect time to get all of this straight, so we can continue on to more interesting stuff