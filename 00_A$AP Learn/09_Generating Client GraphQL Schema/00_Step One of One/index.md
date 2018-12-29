### npm run update-schema

As explained in the "GraphQL Data API" chapter, Relay framework assumes that at design time the GraphQL schema is put together on the GraphQL server and then a relevant portion of it is made available to GraphQL client developers. On the client, the `schema.graphql` file is used by the client-side Relay JS packages.

To generate `schema.graphql`, in `node-dev` container's shell:

```
npm run update-schema
```

As per `package.json` `scripts`, the `update-schema` command is run, which executes `scripts/updateSchema.js`. Per `scripts/updateSchema.js`, the resulting schema extract goes to `src/schema.graphql`.

Distribute `src/schema.graphql` to the Relay JS clients. Should be repeated whenever anything changes in the server's GraphQL schema. 
<br>
Literally, that's all. We are done. Congratulations!