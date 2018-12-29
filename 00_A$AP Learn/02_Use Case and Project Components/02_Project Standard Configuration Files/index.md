### Demo Project Configuration Files

In the project root, there are several files with names starting with a *dot*. This is the common naming convention for configuration files in NodeJS projects. If you develop on Windows, use PowerShell to create or rename "dot" files.

Let's briefly review each one to understand their general purpose:

### .babelrc 

Controls the conversion of modern style JS into the syntax that NodeJS can work with - by the transpiler/compiler tool called `babel`. Often times, this file is copied from another project or a "boilerplate", so it may contain some unnecessary extra stuff. As long as your project installation and runtime work, you can leave [Babel](https://babeljs.io/docs/en/config-files) education to a later stage in your JS developer career 

### .eslintrc.json 

*Linting* is a *static code analysis* process that is supposed to help enforcing proper coding style and architecture. That is a wishful thinking for JS, so JS linting ends up being a code format controlling tool. It sets up rules on indentation, use of semicolons after sentences (remember, those are not required by the JS syntax rules - use them if you like them; the demo project does use them), etc. Remember, consistent formatting is key for Git-managed projects, so linting is very important, just don't overestimate its power beyond formats.

Similar to `.babelrc`, this config file is usually borrowed and reused. Make sure your local IDE is configured to use the "linting" provided with the project, which VSC usually does automatically or after a confirmation prompt. 

Code styling recommendations change all the time, do not get hung up on it - you can always change your style on your next project / assignment. Just keep things consistent in the given scope.

### .gitignore 

Once more, you usually get this file from somewhere and review that it covers components specific to your stack and dev environment. This file is critically important as it's your one and only tool to ensure that neither proprietary, secret information nor your local environment configuration and test output garbage get uploaded into the *remote* Git repository. 

Entires in this config list files and folders that are present in your local project directory but should be excluded from the Git project scope. [Git documentation](https://git-scm.com/docs/gitignore) helps understanding the patterns if you need to cover complex cases. 

Specifically for NodeJS projects, you need to exclude `node_modules/`. When using VSC IDE, put `.vscode` into `.gitignore`. Note that `.git` folder is automatically excluded.
<br>
We'll review `.default.env` in the "Project Custom Configuration File" lesson. Now, continuing on to `package.json`