### Setting up .env 

Now that we have the dev environment hot, let's make sure our configuration file `.env` is correct. As explained before, we need to copy the default file into it.

On the host (remember, any file changes are done on the host, via terminal or IDE), copy `.default.env` into `.env`. You can use Copy functionality of VSC built-in Explorer. 

You don't need to change anything in the file at this point (unless you use a different MongoDB setup), just review it.

- `PORT=80` indicates that NodeJS will run on port 80 of the *container*, which is mapped via `docker run` to port `8080` on the host
- `DB_URI=mongodb://172.17.0.1:27017` sets container's NodeJS server access to MongoDB via port `27017` of the docker *host* - `172.17.0.1` is the default IP of the *host* as seen from containers running on the default Docker network. You can see that in `docker inspect node-dev`, `"Gateway": "172.17.0.1"`. Now, host port `27017` is mapped to `mongo` container port `27017`, where the MongoDB instance runs on. This is the classical way of orchestrating multiple services and servers running on a Docker host. Simple and powerful. Note, the database is not secured, so don't put any sensitive info into it and don't expose your host's port `27017` to the open Internet
- `DB_NAME=web_dev` defines the name of the MongoDB database. The database will be automatically created first time we hit the server to use it
 
On to seeding the data!

