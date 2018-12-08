### Seeding Data

People make a living developing test data generators. Here, we'll use a primitive seeding method: we'll have some data typed in into YAML files and then run a program to load the data into our dev instance of MongoDB

## Review YAML Files

In the IDE, check out `src/data-seed/sample-data/item.yaml` and `user.yaml`. Note, we're hardcoding the `_id` so that we can use these IDs in sample GraphQL queries and mutations delivered with the demo. When seeding development, you'd leave the `_id` assignment lines out, just make sure to keep the dash separating the objects per YAML List format:
```yaml
- _id: 1GVl21lq0S8p
  desc: >-
    Sample Consumer Item
```
would become
```yaml
- desc: >-
    Sample Consumer Item
```

`>-` is a YAML indicator that is used for open text info.

Note `item_details` have a different structure in the two sample items - to illustrate how MongoDB can handle this.

## Review The Loaders 

Net, using the IDE, review `src/data-seed/item-seed.js` and `user-seed.js`. 

In `async function startRun()`, we plug in into the core app's code to initiate the config and connect to MongoDB. This part of code is basically borrowed from `src/server.js`. Once connected, we call `await loadData()` and then exit, closing the DB connection.

`async function loadData()` reads data from the YAML file with the hardcoded name, converts into JS object by using `js-yaml` external 3rd party package, loops over the object's records and creates DB Documents, one by one. Note a particularly efficient way, but simple enough to load a few records. One area for improvement - batch up Create operations, which is achievable by using *bulk* operations, but requires some tweaking, which is out of scope for this course. If interested, you can check out this sample in the [EXLskills gql-server](https://github.com/exlskills/gql-server/blob/master/src/data-load/maintenance-and-conversion/card-interaction-set-course-item-ref.js)

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
```

There may be a few-seconds delay after launching the command till the detailed debug logging fills the screen, but overall the process completes fast.

Yes!!! We've got it kicking

## Check Out The Database Via Compass

Start MongoDB Compass on your host and connect to the `localhost` port `27017` database, with no Authentication, as the entry screen offers as the default. You'll see `web-dev` database. Open it up and start clicking around. It's fun!
