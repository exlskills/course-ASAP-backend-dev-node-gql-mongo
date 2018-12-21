### Setting Up Local .env Configuration File

Now that we have the development environment hot, let's make sure our local configuration file `.env` is set up correctly. 

As explained before, we need to copy the default configuration file provided in the Git project into it. In the host shell:

```
cd C:/docker-vol/demo-gql-mongo
   or
cd ~/docker-vol/demo-gql-mongo
   then
cp .default.env .env
```

You can also use Copy functionality of Explorer in your local VSC IDE. 

Unless you use a different MongoDB setup from the one described in the previous chapter, you would not need to change anything in the `.env` file, let's just review it:

- `PORT=80` - indicates that the HTTP server in NodeJS will run on port 80. As you recall, *container* port 80 in the `docker run` command is mapped to port `8080` of the host
- `DB_URI=mongodb://172.17.0.1:27017` - sets the MongoDB URI for the NodeJS application. From inside the NodeJS container, the path to MongoDB is via port `27017` of the docker *host*.
 In the "Docker Networking ..." lesson you learned that `172.17.0.1` is the IP of the *host* on the default Docker network that both of our containers run in. How do we know the containers are running in the *default* Docker network? First, because we haven't configured any additional Docker network on our host. Second, in the `docker inspect node-dev` output, `"NetworkSettings"`, you can see the details, e.g., `"Gateway": "172.17.0.1"`. Now, on the MongoDB container side, local port `27017` is mapped to port `27017` on the host. So, host's port `27017` in effect is the port that our dev MongoDB is listening on.
 This is the classic way of orchestrating multiple services and servers running on a Docker host. Simple and powerful. 
 As a disclaimer, the database is not secured, so don't put any sensitive info in it and don't expose your host's port `27017` to the open Internet. Even if you configure this DB with a password, you'll end up saving it in the open in the `.env` file. So, don't bother, just don't keep anything confidential in your dev DB. As we discussed in the "Project Custom Configuration File" lesson, the Production environment is well protected by virtue of running your Docker Production containers and services inside a secured orchestration framework. In dev, it's all on your laptop - with the intent that you can develop efficiently. Unless you know how to absolutely secure access to your laptop - don't copy your production database down into it to seed the development. Not a good practice.
- `DB_NAME=web_dev` - defines the name of the MongoDB database. The database will be automatically created if it doesn't exist - the first time we hit the DB server loading data into it
 

Next, on to the `install` step in our dev project lifecycle!