### Structure of the JS Code Base 

`server.js` located in the root code folder `src/` is the entry point into the application. All other code components are placed into sub-folders, except `schema.js`, which is used by the GraphQL engine. Technically, the schema file could have been placed into a sub-folder as well: if you run a search for where the file is used, you'll see this import statement in `server.js`
```
import { Schema } from './schema';
```
If the `schema.js` file is placed into a different location, the import would need to be adjusted.

There is also the `schema.graphql` file in the `src/` folder. This is a generated file that should be used in the GraphQL client, will explain this in the GraphQL chapters.

## Demo Project JS Code Folders 

- `src/config` - initiates runtime configuration objects with values coming from OS Environment or values from the `.env` file, or hardcoded defaults. The folder also contains sample SSH key files to handle the JWT security - optional feature commended out in the demo. JWT is discussed in later chapters
- `src/data-seed` - code and sample source YAML files to pre-load the database with test data
- `src/db-handlers` and  `src/db-models` - the data layer, hardcoded for the use of MongoDB via the `mongoose` package, reviewed in details in the designated chapter. Isolating the data layer into a separate set of objects is a common design strategy - helps switching to a different database if needed, by re-writing only the data layer components
- `src/http-middleware` - term `middleware` is widely used in JS/web development and can be applied to any program layer that handles any kind of transformation flows and processes (or anything at all). Typically, middleware would be a pluggable, isolated layer, enabling coordination between integrated components. Accepting whatever terminology is used by the dev community, this `http-middleware` component runs the flow from the http request coming in to the point where the "controller"-like logic (business rules) takes over. In the demo, this component is included to show its potential use but it is bypassed in the coded flow.
- `src/paging-processor` - this is a set of sample code that enables the basic generic navigation through datasets via "paging" request, like "extract next n elements after element x" or "last n before x", etc. The code is used in conjunction with the MongoDB and GraphQL functionalities (so, it is a "middleware")
- `src/relay...` folders - facilitate the data handling layer according to the GraphQL standards. Discussed in great details in the corresponding chapter
- `src/utils` - miscellaneous code objects, including `logger.js` responsible for facilitation log generation

A dev familiar with the MVC design would be wondering where are the proper Models, Views and Controllers? They are kind of embedded in the GraphQL core of the above design, just packaged closer to the data itself. Will talk about it more in the GraphQL chapter.

And now - on to the MongoDB stuff
 
