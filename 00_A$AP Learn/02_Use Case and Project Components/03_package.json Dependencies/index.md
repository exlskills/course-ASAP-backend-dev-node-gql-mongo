### package.json and package-lock.json

Just to make sure, you probably realize that in JS, the concept of *package* relates to an entire business or utility application vs. a *folder* as in a Java application. Subsequently, the plentiful JS 3rd party utilities are called *packages* - they are *packaged* to be used elsewhere.

Likewise, a NodeJS project must be *packaged* for the NodeJS software to run it. Getting the project to a runnable state usually takes a few steps in the project's *lifecycle*. `package.json` is the place where all project information and configuration is stored in. This file is the heart of the NodeJS project. It is written to [NodeJS Package Manager standard](https://docs.npmjs.com/creating-a-package-json-file). Sure enough, the file is also usually copied from somewhere initially and then updated, although, you may see `npm init` mentioned as the method of creating `package.json` from scratch - corresponding to the project initiation step in its lifecycle.

`package-lock.json` is a control file that is auto-generated (or updated) when the Package Manager `npm` runs the project installation step and brings in all required 3rd party ("external") packages locally, as listed in the `dependencies` and `devDependencies` sections of `package.json`. Just like with `Maven` in Java or `pip` in Python, proper dependencies management is absolutely crucial for the wellbeing of your project. 

Stable release versions of external packages are usually three dot-separated numbers, e.g., `8.1.12`. The dependency list specifies the version to be loaded by the Package Manager into `node_modules/` and used at runtime. As the external packages have their own dependencies, the Package Manager applies some logic to ensure that the entire required dependency tree is brought in. Reusable common dependencies and everything listed in your `package.json` is placed into individual sub-folders under `node_modules`. If an external package requires a version of some package different from the one listed in `package.json`, that version will be loaded into its own `node_modules` sub-folder under the external package's sub-folder.

*Version-matching* rules are used to advise the Package Manager on what leeway does it have selecting what version to bring in. You may ask for the exact 3-pos "Patch" version or for any Patch within the 2-pos "Minor" release, or for anything in the 1-pos "Major" release. The rules can be coded using `< = >` signs and/or tilde or caret `~^` in front of the numbers. Asterisk `*` means "any version", including *unstable*; word `latest` means "the latest available stable version" of the package. The tilde `~` generally means the exact match of the listed numbers. The caret `^` can be confusing as its logic is based on zero vs. non-zero numbers in the combination it prefixes. 

In the most common scenario, when adding an external package to your dependencies list, the version is set to the 3-pos latest available stable one and a caret `^` is placed in front it. This guarantees that the package will be brought in at that or later version, but still within the same Major - no auto jump to later Majors. 

The latest available package version can be found by going to the package's npm site or suggested by the IDE when editing `package.json` (must be connected to the npm system over the internet, of course). `npm install <package name>` described below adds the package to the list automatically.

Once the logic is evaluated and applied by the Package Manager, the resulted combination of packages with versions is written into the *lock* file `package-lock.json` (or `yarn-lock.json` if using `yarn` vs. `npm`). The presence of the lock file guarantees that the Package Manager will load the exact same dependency tree as described in it, therefore creating repeatable installations with matching content of `node_modules/`.

The `package-lock.json` gets updated when you change the `package.json` (or delete the lock file).

So, the question is what logic should you apply when defining external package version requirements? Basically, if you are too strict on the versioning (require the match at the Patch level) - later in the lifecycle of your project, if you change some external packages, you may start falling behind on versions of mutual dependencies, and the Package Manager will have to bring external requirements into separate `node_module` sub-folders, per package, vs. using the project-wide ones. This will grow the size of your project's footprint. If, as the other extreme, you allow loading the *latest* versions of your dependencies, there is a risk that when the lock file is updated - you'll get versions of external packages loaded that break your code.

You may read discussions on how important it is to periodically refresh dependencies to avoid the *code stagnation*. Well, it is like continuously re-writing your own code when you find out about some method or feature you haven't used before. Use it on your *next* project. Well-working code should be left alone, till it is necessary to change it - usually due to legitimate business reasons vs. shifts in coolness trends. Vulnerability issues aside - as explained below.

Different systems periodically inform you if vulnerabilities have been identified in the external packages your project has loaded. You will need to review your `package-lock.json` to see where and how the *bad* package is used and then fix the underlying cause in `package.json`. Otherwise, you would probably never change versions after the initial development, till you're ready to upgrade the entire project. 

Packages listed in the `devDependencies` section are only required for the dev environment and are not loaded when the production environment is set up.

We will review the content of `package.json` used in this course in details when we look at the individual JS code components and processes

### Adding New Packages Via the Command Line 

In any npm package documentation, you'll see `npm install <package>` as the way to use it. This is a convenient way adding a new package to you project: the package will be installed at the latest stable version and added in the alphabetically sorted location in `package.json` dependencies, with the `^` prefix in front of the version. To add new packages to `devDependencies`, use `npm install <package> --save-dev`

Everything you do via `npm`, by default affects the current directory only. However, a "global" install of packages is also available. It is a common recommendation *not* to install packages globally for various reasons. When developing in a dockerized environment - a global install is not even an option, as the container running NodeJS would only see project's local `node_modules` folder.
<br>
Next, we'll review the `scripts` section of `package.json`