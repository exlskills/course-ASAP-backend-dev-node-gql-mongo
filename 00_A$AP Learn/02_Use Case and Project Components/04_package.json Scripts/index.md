### package.json scripts Section 

Similar to `Makefile` or `Maven`, `npm` promotes the concept of NodeJS project lifecycle *stages* and their *"automation"*. Some stages are `init`, `install`, `start`. As we agreed, we skip the `init` stage as all projects these days come from some prototype projects. The `install` stage is pretty key: `npm install` loads the dependencies. 

The `scripts` section in the `package.json` enables configuring custom behavior associated with the stages. Obviously, in the era of sophisticated configuration management tools such as `Ansible`, `Chef`, `Pappet`, `Salt`, managing project via scripts sounds somewhat basic, so the `scripts` approach should be taken at its face value and used to handle small specific tasks. 

Let's review the content of the `package.json` `scripts` section in the demo project:

```json
"scripts": {
    "start": "npm run start:dev",
    "start:production": "better-npm-run start-prod",
    "start:dev": "better-npm-run start-dev",
    "update-schema": "babel-node ./scripts/updateSchema.js",
    "prettier": "prettier --write \"{*.js,!(node*)**/*.js}\"",
    "build": "babel src -d build --copy-files && npm run build:copy",
    "build:copy": "copyfiles package.json ./build",
    "lint-fix": "eslint --fix ."
  },
  "betterScripts": {
    "start-prod": {
      "command": "node build/server.js",
      "env": {
        "NODE_ENV": "production"
      }
    },
    "start-dev": {
      "command": "nodemon -w src/server.js --exec \"babel-node src/server.js\"",
      "env": {
        "NODE_ENV": "development"
      }
    }
  },
```

The left side represents the name of the "state" or the "command" to be executed via `npm run`, e.g., `npm run update-schema` or `npm run build`. Note that `npm` has several standard *CLI commands* which are run by placing the command directly after `npm`, e.g., `npm install`, `npm start` - but we have the `start` script in the section as well. 

Placing or omitting the `run` in `npm` seems unnecessary confusing and controversial. Generally, `npm run <script name>` executes just the script, whereas `npm <command name>` runs the full logic associated with the command. `npm` documentation has a full page write up on how individual commands line up against scripts with the same name. Luckily for us and most developers, one would never use this stuff, other than the `start` override, which in our case works fine via `npm start`. Our `start` script acts as an *override* to what the standard `npm start` would do, which is kicking off `node server.js`.

This is the custom script that gets executed via `npm start`:

```
"start": "npm run start:dev"
```

Our `start` script runs another script named `start:dev`, which is also present in the `scripts` section. In turn, `start:dev` runs `better-npm-run start-dev`. `better-npm-run` is a package used to facilitate scripts execution on Windows - it covers the file path differences (`/` vs. `\`). `better-npm-run` is listed in `dependencies`, and the `start-dev` is in the `betterScripts` section. 

Another popular package used on the Windows platform is `cross-env`. As we pointed out, we do *not* run NodeJS in Windows, even if we develop on a Windows laptop: NodeJS runs in a Linux container. However, using compatibility packages in NodeJS scripts is a common approach.

`nodemon` is also a package listed in `dependencies`. It "watches" the `src/server.js` file for changes and restarts the process as specified via the `--exec` option, which in our case is `babel-node` execution of the `src/server.js` file, the entry point into the application. Some more on the `nodemon` watcher below.

As you can see, `start-prod` executes the code from the `build` folder, directly from `node`, so that code must be processed via `babel` ahead of time - which is done, according to the scripts, in the `build` step.

This is a typical flow, but there are other ways and packages to use in either dev or prod flows. The options and packaging change, so often times `package.json` would contain components that are redundant or not even used in the actual implementation. The downside is a larger footprint and longer times building an instance, so, technically, `package.json` should be periodically optimized, but such an optimization is definitely not on the critical path to getting your application to work.

Will review `update-schema` in the GraphQL chapters. `prettier` and `lint-fix` are examples of helper scripts to manage linting and formatting in bulk, outside of the IDE.

### Note On nodemon Watcher

The auto-restart option sounds like a convenient feature that increases development productivity: once some code changes take place, the server restarts by itself, and the developer can see if the changes worked or not. As the IDE should be set to the auto-save mode, edits get flushed to the disk in near real-time (although, there are ways to configure auto-safe with delays or do it on file exit, etc.). So, in reality, the server will be continuously restarting while a series of related edits is being coded. Eventually, the last restart is supposed to be the one reflecting the desired final state of the code. 

Keeping the server restarting with incomplete code is Ok - it should not break anything. The problem is that `nodemon`-`babel-node` combination appears to have issues when working in a container. On Windows, host-managed file changes are not triggering the *watcher* functionality in the container, so the method doesn't work at all. When the container does see the changes, the process fails to close the running server before starting it again or works intermittently. 

So, it looks like for the time being, till the tools catch up, manual restart of the server in the container shell is the only sure option: `Ctrl-C`, then `Up Arrow` to bring `npm start` back, followed by `Enter`. 
<br>
Next, we'll review how the environment configuration is set up and maintained in the demo project