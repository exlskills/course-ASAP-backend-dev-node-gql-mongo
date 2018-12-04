### package.json scripts Section 

Similar to the concept of `Makefile` or `Maven`, `npm` (or `yarn`) enables "automation" of the project lifecycle stages. Here comes `npm install` that loads the dependencies. The idea behind the `scripts` section in the `package.json` is that the dev can configure some custom behavior associated with those stages. In the era of sophisticated configuration management tools such as `Ansible`, `Chef` and `Pappet`, managing your project via little scripts sounds somewhat naive, so the dev should be taking the `scripts` approach at its face value and use to handle small specific tasks. 

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
      "command": "nodemon -w src --exec \"babel-node src/server.js\"",
      "env": {
        "NODE_ENV": "development"
      }
    }
  },
```

The left side represents the name of the "state" or the "command" to execute via `npm`. Note, that `npm` has a list of standard commands which are run by placing the command directly after `npm`, e.g., `npm start`. So, here we have a script that is executed when `npm start` is run:
```
"start": "npm run start:dev"
```

You see that the script runs another script named `start:dev`, which is present in the `scripts` section. Notice the `run` addition in between `npm` and the name of the script: running custom scripts requires this `run` keyword, unlike running standard scripts/commands. Somewhat confusing - why not to use a uniform syntax always? Although, `npm run start` in our case acts similarly to `npm start`, `npm run install` fails with `missing script: install`. So, you get the idea: if the script is present, you can add `run` to execute it even if the script overrides a standard command (if you're not sure whether the command is standard or not), and you must add `run` to execute custom ones. 

`start:dev` in our setup runs `better-npm-run start-dev`. `better-npm-run` is a package used to facilitate scripts execution on Windows - it covers for the file path differences (`/` vs. `\`). `better-npm-run` is listed in `dependencies`, and the `start-dev` is in the `betterScripts` section. `cross-env` is another popular package used on the Windows platform.

`nodemon` is also a package listed in `dependencies`. It "watches" the `src` directory for changes and restarts the process as specified via the `--exec` option, which in our case is `babel-node` execution of the `src/server.js` file, the entry point into the application. 

As you can see, `start-prod` executes the code from the `build` folder, directly from `node`, so that code must be processed via `babel` ahead of time - which is done, according to the scripts, in the `build` step.

This is a typical flow, but there are other ways and packages to use in either dev or prod flows. The options and packaging change, so often times `package.json` would contain components that are redundant or not even used in the actual implementation. The downside is a larger footprint and longer times building an instance, so, technically, `package.json` should be periodically optimized, but such an optimization is definitely not on the critical path to getting your application to work.

Will review `update-schema` in the GraphQL sections. `prettier` and `lint-fix` are examples of helper scripts to manage linting and formatting in bulk, outside of the IDE.
