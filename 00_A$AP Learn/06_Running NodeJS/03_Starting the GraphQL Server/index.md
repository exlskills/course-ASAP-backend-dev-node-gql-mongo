### Starting the Server Flow

If you are not in the `node-dev` shell yet, get in there: from your host's terminal shell (PowerShell on Windows), enter `node-dev` shell:
```
docker exec -it node-dev bash
cd /myapp
```

We'll start the server via the `npm start` command that has an override in `package.json` to go through `babel`. This was discussed in the "Use Case and Project Components" chapter.

So,
```
npm start
```

After a brief hesitation, you'll see `info` and `debug` messages coming from the `logger` commands and `Mongoose:` message from the `mongoose` debug logging. The process runs in the `node-dev` bash shell, so to stop it you do `CTRL-C` while in the shell. You can open multiple shell sessions by launching `docker exec -it node-dev bash` from your host's terminal shell, in case you'd like to check something in the running container while your first shell is occupied by the NodeJS server process. In the dev mode, you do want to run NodeJS "interactively" so that you can see the logs as you test functionality via a browser. Multiple displays would come handy.

You can check `docker stats` on the host to see how much resources the containers take up. Not bad at all! Neither NodeJS nor MongoDB are resource hogs, thankfully.

### Quick Test from the Browser

On the host, open a browser and navigate to `http://localhost:8080/graph` (change the `8080` port to the one you used in the `docker run` launching `node-dev`, if it was different)

You'll see a `GraphiQL` work screen. Great, everything works!
