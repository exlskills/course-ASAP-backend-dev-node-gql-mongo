### Using External Code Components in JS

In JS, any objects, methods or variables used in a code file but not defined in the file itself, must be *explicitly* imported before they can be used. This would be true for Java, Python and GoLang as well, so no surprise in this respect. 

The `from` part of JS *import* statements refers to either an external *package* listed in `package.json` `dependencies`, or to a *file* somewhere in the project. In the latter case, the path to the file must be provided relatively to the location of the file the import is *coming* to - the file you're writing the `import` statement in. 

Unlike in Java, where each code component is defined using a dotted path of folders (packages), starting from the code root, in JS code imports use Unix-like file system paths. E.g., in `src/relay-mutations/user/create-user-mutation.js`, you can see the following import statements:

```
import { CompletionObjType } from '../../relay-models/completion-obj-type';
import { createUser } from '../../relay-mutate-and-get/user-mag';
import { logger } from '../../utils/logger';
```

Matching to the file system tree under `src/`, the relative path to get from the destination 

```
src/relay-mutations/user/create-user-mutation.js
```

to the source

```
src/relay-models/completion-obj-type
```

is to move two sub-folder levels up to `src/` and then down into `src/relay-models`: 

```
../../relay-models/completion-obj-type
```

The extension `.js` is omitted in imports. 

Imports may also come from an `index.js` file placed into a sub-folder. In that case only the name of the sub-folder (with the path) should be provided , e.g., 

```
import config from '../config';
```

actually comes from `src/config/index.js`.

Only explicitly *exported* code components can be *imported* into other code files. 

Export and import rules and syntax have evolved from early versions of JS. In this course we're committed to using modern JS, so we avoid the old style `require` in favor of `import` - anything that makes JS code look as other professional languages developers are familiar with. 

As the ultimate use of imports is to bring in *exported* code, the key to understanding what is *imported* is actually on the *export* side. Let's look at `src/data-seed/user-seed.js`:

```
import mongoose from 'mongoose';
import * as fs from 'fs-extra';
import path from 'path';
import * as yaml from 'js-yaml';
import User from '../db-models/user-model';
import config from '../config';
import { logger } from '../utils/logger';
``` 

When using external packages - follow the documentation. However, even though `require` is a thing of the distant past, it remains the only documented mode in many packages. E.g., `js-yaml` documentation says to use

```
yaml = require('js-yaml');
```

A direct replacement of `require` with `import * as` or just `import` usually works:

```
import * as yaml from 'js-yaml';
```

The short version, without the `*`, is used in the `mongoose` import:

```
import mongoose from 'mongoose';
```

Digging deeper, however, 

```
import * as <name> from <source>
``` 

is not the same as  

```
import <name> from <source>
```

Again, in those two `import` forms, the real outcome is determined by what is on the *export* side. The asterisk `*` in the `import` means "bring everything that has been exported", whereas the construction without the `*` imports only the so-called *default* export - identified in the source as `export default`. Traditional old-style 3rd party packages use `export default` exclusively and do not have any other (*"named"*) exports, therefore, importing with or without the asterisk from those packages brings in the same stuff.

On the export side, the modern recommended approach is to use "named" vs. *default* exports. E.g., in `src/utils/logger.js`, we have:

```
export const logger = winston.createLogger({
```

vs. 

```
export default winston.createLogger({
```

Accordingly, modern 3rd party packages use named export - a separate one for each exported component. This allows importing needed components only vs. the entire content of the package. The `import` statement lists the components in curly brackets (`{}`). E.g., in `src/relay-models/item-price-type.js`:

```
import { GraphQLFloat, GraphQLObjectType } from 'graphql';
```

The downside of the above - extra typing. The IDE may help auto-generating imports and it will validate object names, but often times it's the developer's job to type the names in. 

Missing imports break your code execution. Extra imports slow down the program load. For the backend dev, it looks like the program load speed is a minor consideration, compare to the front end scenario where all imports must get physically loaded into the browser client over the Internet and compiled before the app code flow really starts.

The demo provides examples of `import` and `export` usage that come from various sources - this would be the case in many production JS projects, though. E.g., 

- `mongoose` models came from traditional examples, and they use `export default`
- GraphQL examples show how to chain `export` and `import` from multiple files and folders using intermediate `index.js` files
- `server.js` has `require` in a construction that would need to be considerable re-written to use `import` instead:

  ```
  graphQLApp.use('/healthcheck', require('express-healthcheck')());
  ```

JS syntax and styling considerations evolve, just pick something you feel comfortable with - you can always change it working on your next project (and you very likely will). And then change again.



After the `import`/`export` warmup, next, we'll review the demo project JS code components