### npm install

As a reminder, in order to load the required npm packages your host should be connected to the Internet. `npm install` is run in the project folder in the `node-dev` container shell (where we've got NodeJS and npm as we're using the official NodeJS image for the container). To get into the `node-dev` shell, from your host's terminal shell (PowerShell on Windows), type:
```
docker exec -it node-dev bash
```
Then switch to the folder that is mapped to the host's project folder:
```
cd /myapp
```

All set, now run `npm install`, which takes a minute or two to complete. The warning are generally ignorable. If you see vulnerabilities reported, please enter an issue in the [demo GitHub project](https://github.com/exlskills/demo-gql-mongo/issues).

As this is just a demo/dev run in an isolated local environment, npm vulnerabilities should not compromise your box.

As a recap of what we learned earlier, `npm install` is a standard `npm` command that reads `package.json` and `package-lock.json` (if present), evaluates which external package dependencies should be installed and loads them into `node_modules` folder in the project directory. `package-lock.json` is created/updated if anything in `package.json` changed since the last `npm install` run. Every time you update `package.json` you should re-run `npm install`, and upon significant updates or package version changes, you should be deleting `node_modules` folder and `package-lock.json` to allow `npm` to reprocess the dependencies tree from scratch. However, changes in the tree may cause your code to break, so test thoroughly. Luckily, you always have your last working version and a full changes history available in the GitHub project to fall back to. What a wonderful dev world!