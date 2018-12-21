### Seeding Test Data

People make a living developing test data generators. Here, as a poor man's seeding method, we'll type in some data into YAML files and run a few simple custom programs to load the data into our dev instance of MongoDB.


## Review data-seed YAML Files

In the IDE, review `src/data-seed/sample-data/item.yaml` and `user.yaml`. Note, we're hardcoding the `_id` so that we can use them in the sample GraphQL queries and mutations delivered with the demo. When seeding actual development, you'd leave the `_id` assignment lines out. 

In case you don't know much about YAML other than everything must be indented and list elements must start with a dash, `>-` is a YAML indicator used in front of open text info.

Notice that `item_details` have different structures in the two sample Items and User Order Items as well - to illustrate how MongoDB handles flexible structure Documents.


## Review data-seed Loaders 

Next, using the IDE, review `src/data-seed/item-seed.js`, `user-seed.js` and `user-order-seed.js`

In `async function startRun()`, we're plugging in into the code of the core application so that we can utilize the same configuration and connect to MongoDB. This part of the code is, basically, borrowed from `src/server.js` that we'll review later. Once connected to the DB, we call `await loadData()` and then exit, closing the DB connection.

`async function loadData()` reads data from the YAML file with the hardcoded name, converts into JS object with the help of `js-yaml` external 3rd party package, loops over the object's records and creates DB Documents, one by one. Not a particularly efficient way, but simple enough to load a few records. One area for improvement - batch up Creates, which is achievable by using *bulk* operations in MongoDB. This would requires some tweaking, which is out of scope for this course. If interested, you can check out [this sample in the EXLskills gql-server](https://github.com/exlskills/gql-server/blob/master/src/data-load/maintenance-and-conversion/card-interaction-set-course-item-ref.js)


## Run the Loaders

From your host's terminal shell (PowerShell on Windows), enter `node-dev` shell:

```
docker exec -it node-dev bash
cd /myapp
```

Then copy - paste the load launch commands from the top of the `.js` files and execute them from the `node-dev` shell:

```
npx babel-node src/data-seed/item-seed.js
npx babel-node src/data-seed/user-seed.js
npx babel-node src/data-seed/user-order-seed.js
```

There may be a few-seconds delay after launching the command till the detailed debug logging starts filling the screen, but, overall, the process completes fast.

Yes!!! We've got it kicking.


## Review The Data in MongoDB Compass

Start MongoDB Compass on your host and connect to the `localhost` port `27017` database, with no Authentication - the default offer of the entry screen. You'll see `web-dev` database. Open it up and start clicking around. It's fun!


We can start the GraphQL server now, let's do it!