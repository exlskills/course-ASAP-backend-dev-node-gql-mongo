### Using External Code Components in JS

In JS, any objects, methods or variables used in a file, which are not defined in the file, must be imported - this approach is common for Java, Python and GoLang as well. The `from` part of import statements refers to either an external package listed in `package.json` `dependencies`, or to a specific file in the project. The path to the imported code must be provided relatively to the location of the file the import is coming into. Unlike Java, where each code component is defined via the dotted combination of folders, starting with the root one, in JS project code imports follow Unix-like file system path designation. E.g., in `src/relay-mutations/user/create-user-mutation.js`, you can see the following import statements:
```
import { CompletionObjType } from '../../relay-models/completion-obj-type';
import { createUser } from '../../relay-mutate-and-get/user-mag';
import { logger } from '../../utils/logger';
```
Matching to the file tree, `../../relay-models/completion-obj-type` is two sub-folder levels up to get to the code root (`src/`), and then down into `src/relay-models` where `completion-obj-type.js` is located. The extension `.js` is omitted in imports. You can also import from `index.js` file placed into a sub-folder, providing only the name of the sub-folder (with the path) and omitting `index.js`, e.g., `import config from '../config';`

Only components explicitly exported in the code can be imported and used in other code files. As the export and import rules and syntax have evolved throughout the versions of JS, it may get quite confusing seeing `require`, `export`, `export default`, `import` and `import as` statements in programs and samples. As in this course we're committed to using the latest version of JS, we avoid the old style `require`, which are generally equivalent to the modern `import`. The goal is to have our JS code looking as close to other languages we're already familiar with, and `import` is a sure winner using this approach. 

Now, what about the curly brackets and asterisks in this code from `src/data-seed/user-seed.js`?
```
import mongoose from 'mongoose';
import * as fs from 'fs-extra';
import path from 'path';
import * as yaml from 'js-yaml';
import User from '../db-models/user-model';
import config from '../config';
import { logger } from '../utils/logger';
``` 
The key is in knowing what and how is exported in the code files from which the import takes place. For external 3rd packages, the documentation always explain the usage, however, more often than not it only shows how to `require` the package vs. importing it. In majority of cases, a direct replacement of `require` with `import` statement works. E.g., `js-yaml` documentation says to use
```
yaml = require('js-yaml');
```
And we replaced it with 
```
import * as yaml from 'js-yaml';
```
We could have used a shorter version, without the `*`, as seen in the `mongoose` import:
```
import mongoose from 'mongoose';
```
Technically, `import * as <name> from <source>` is different from `import <name> from <source>`, but only in case the `<source>` has multiple exports. `*` means "import everything", whereas the construction without the `*` imports only the so-called default export (identified in the source as `export default`. Traditional old-style 3rd party packages use `export default` and do not use multiple "named" exports, so both constructions do the same thing.

The recommended approach is use "named" exports, e.g., in `src/utils/logger.js`, we write:
```
export const logger = winston.createLogger({
```
vs. 
```
export default winston.createLogger({
```

Modern 3rd party packages use named export all the time, e.g., the `graphql` package used extensively in the demo. Therefore, imports from `graphql` list individual objects (methods) in curly brackets (`{}`), e.g., in `src/relay-models/item-price-type.js`:
```
import { GraphQLFloat, GraphQLObjectType } from 'graphql';
```

The downside of this is extra typing. The IDE may prompt you and auto-generate imports when you refer to external components in your code or copy/paste from other file in your project. 

Missing imports break your code execution. Extra imports slow down the program load. For the backend dev, it looks like the program load speed is a minor consideration, compare to when your JS code runs in the browser client (where all the code must be loaded into via the internet and compiled by the browser before the execution).

What approach should you use? Like in many similar situations - starts using the code, you'll figure out what works best for you. The demo gives you examples of usage that come from various sources. E.g., `mongoose` models that came from traditional examples use `export default`; GraphQL examples show how you can chain the export and import from multiple files and folders using intermediate `index.js` files. `server.js` even has an exapmple of using `require` in a construction that would need to be considerable re-written to use `import` instead:
```
graphQLApp.use('/healthcheck', require('express-healthcheck')());
```

JS syntax and styling considerations evolve, just pick something you feel comfortable with - you can always change it writing your next project (and you very likely will)

