### Doing npm install

As a reminder, in order to load the required npm packages your laptop should be connected to the Internet. 

We talked about `npm install` in the "Use Case and Project Components" chapter. It is run in the system that has NodeJS and `npm` software present, which is the `node-dev` container. So, we need to get into the container's shell - we tested its shell access in the previous chapter, after starting the container.

In your host's terminal shell (PowerShell on Windows), type:

```
docker exec -it node-dev bash
```

This puts you into the container's shall. Then switch to the folder that is mapped to the host's project folder:

```
cd /myapp
```

All set, now run:

```
npm install
```

The process which takes a minute or two to complete. Much longer on slow connections and weaker hardware.

Conveniently, the process displays its status and produces some output as well. The warning are generally ignorable. If you see vulnerabilities reported, please enter an issue in the [demo GitHub project](https://github.com/exlskills/demo-gql-mongo/issues). Although, npm vulnerabilities would unlikely compromise your box when running dev/demo in an isolated local environment, with no access to your serving port (8080) from the Internet.

As a recap of what we learned earlier, `npm install` is a standard `npm` command that reads `package.json` and `package-lock.json` (if present). It evaluates which external package dependencies should be installed and loads them into the `node_modules` folder in the project directory. `package-lock.json` is created (or updated if anything in `package.json` changed since the last `npm install` run). Every time you update `package.json`, you should re-run `npm install`, and upon significant updates or package version changes, you should be deleting `node_modules` folder and `package-lock.json` to allow `npm` to reprocess the dependencies tree from scratch. However, changes in the tree may cause your code to break, so test thoroughly. Luckily, you always have your last working version and a full changes history available in the GitHub project to fall back to. What a wonderful dev world!

As `node_modules` folder is created and loaded by the container-run process, in Linux, your laptop user will not have access to it without sudo. This is somewhat inconvenient when you want to check external packages out from your IDE. If that is the case, you can change the ownership of the loaded folder, of course. Or, as mentioned earlier, run the container under a non-root user.


On to seeding the test data!