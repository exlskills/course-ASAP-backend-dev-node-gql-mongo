### What Is In The Query Resolver

A *Resolver* is an integral part of the GraphQL query definition schema. E.g., in the `src/relay-queries/user/user-list-query.js`, we have:

```
resolve: (obj, args, viewer, info) =>
    resolveListUsers(obj, args, viewer, info)
```

This is the actual *function* that executes the query on the GraphQL server - like the *Controller* in *MVC*. As you've learned in the coding Elementary School, application code should be modularized, so instead of writing JS commands right in the query definition (as we could have) - we put those in a separate imported file `src/relay-resolvers/user-resolver.js`. 

We've looked inside `resolveListUsers` already when reviewing the pagination implementation. In the demo project, *query resolvers* represent the intermediate layer between the GraphQL query and the database layer. The idea is that if the database layer changes, e.g., away from MongoDB to something else - the GraphQL side code won't need to be changed. Just a nice design principle.

### Query Resolver Arguments

The GraphQL engine passes the four-member `obj, args, viewer, info` argument group into the query resolver. We've detailed each of those already, so just to recap:

- `obj` - a GraphQL engine managed object. Applies to queries that are executed inside *parent* queries, otherwise - null. `listUserOrders` that we review later in this chapter illustrates how this parameter is used
- `args` - this is what the client sends as the *query arguments* 
- `viewer` - this is the GraphQL engine controlled *context* that we configured to be the object identifying the *user* executing the query on the browser client side
- `info` - a raw detailed representation of the query, we don't use it as we have everything we need organized by the engine into other parameters already, but we are passing it just in case we would ever need it for something. At the end of this chapter we mention a potential use case where `info` can be utilized in the process logic.

### Note on Passing Arguments into Function Parameters in JS

In JS (and Java as well), *parameters* are the names written in the function *definition*, and *arguments* are the things written in the *call* to that function. So, that is the theory. In practice, of course, the terms *parameters* and *arguments* are used pretty much interchangeably. One can say, "I'm calling the function passing these parameters into it", and everyone will understand. Although, the correct form would be "I'm calling the function passing these arguments into it".

Arguments in JS are *passed by value*. So, changes to things like numbers or strings are not reflected back in the calling function. This is consistent with Java (*primitives* are passed by value) and generally is the expected behavior. Because the *value* of an *Object* in JS (and Java) is actually a *reference* to the object's content, changes to the *content* of the object are propagated up the call chain. Again, pretty expected, normal behavior. 

Another place where we use the notion of object *value* vs. *content* is when declaring objects as *const* - and then changing the object's content in the downstream code. 

The arguments are matched to parameters *by position*, *not* by name. As parameters do not have *types* declared, there is no any validation on either the type or number of arguments passed into the call. NOTE: TypeScript *does* enforce full-featured declaration and validation of functions, parameters and arguments. That is why it is recommended so much! Put TS on your must-learn and must-use list - soon after completing this course.

### Viewer-based Security and Validation

The resolver is a good place to code a security layer that would control access to the query function as well as cleanse sensitive data out as the DB query returns its result. In the demo project use case, security is not coded. It would be applied based on the content of the *viewer* object. E.g., we can control that a standard User can only see his/her orders, whereas a Shipping Manager can query all orders in his/her line of work. 

Note that as individual fields each have their own *resolver* running after the *query resolver* completion (or the *default* one if none specified in the field definition) - outbound field-level security can be coded in the field resolvers.

### Argument Content Validation

Generally, the schema should be set up with the base query arguments validation built in. E.g., we can use `GraphQLNonNull` to mark required arguments, control the argument type and *enumeration* content. More complex validation involving multiple input arguments can be put into the query resolver

### GraphQLID to Internal ID Conversion

`base64` encoded GraphQL IDs are converted into corresponding internal IDs in the query resolver by executing `graphql-relay` function `fromGlobalId`. The conversion back to the GraphQL ID is best done in the field resolver on the way out
<br>
Next, we are passing through the resolver into the MongoDB query processor code!