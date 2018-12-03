### Configuration Files

A common convention is to name configuration files starting with a dot: `.` On Windows, you have to use PowerShell to create or rename "dot" files.

## .babelrc 

Controls the conversion of modern style JS into the syntax that NodeJS can work with. Often times, this file is copied from another project or a "boilerplate", so it may contain unnecessary extra stuff. As long as the installation and runtime work, you can leave your proper [Babel](https://babeljs.io/docs/en/config-files) education to a later stage in your JS dev career 

## .eslintrc.json 

Similar to the above, this config file is usually borrowed and reused. One thing to keep in mind is to make sure your IDE is configured to use the "linting" provided with the project, which VSC usually does automatically or after a confirmation prompt. 

## .gitignore 

Once more, you usually get this file from somewhere and review that it covers components specific to your stack and dev environment. This file is critically important as it's you one and only tool to ensure that neither proprietary and secret information nor your specific environment setup and test output get uploaded into the Git repository. [Git documentation](https://git-scm.com/docs/gitignore) helps understanding the patterns if you need to cover a more complex case. 

Specifically for JS, you need to exclude `node_modules/` from your git project scope - the folder where standard 3rd party software is loaded when you run `npm install`

Continuing to `package.json`