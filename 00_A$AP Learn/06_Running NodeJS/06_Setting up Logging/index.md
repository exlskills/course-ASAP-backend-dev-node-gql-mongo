### Winston Logger

NodeJS doesn't have a built-in logging framework. The standard beloved JS `console.log` can be used, but there are several (competing) 3rd party solutions that work pretty well.

The demo project uses the `winston` package. As a package must have a name, `winston` can be just as good of a name for a logger as, e.g., `log4j`.

In the past, configuration around logs has been an important component of any server software install process. Logs would be rotated, archived, passed on to a monitoring solution. `winston` has a complete set of capabilities configuring log outputs. 

However, containerization may be the death of old-stile loggers. Basically, the console log output is all we need when running a server in a container, for either dev or production. As Docker reads the console-out, it seamlessly passes it on to the monitors and/or consolidators, bypassing anything that has to do with dedicated log files and their rotation.

So, in the demo project, the use of `winston` is limited to enhancing the console output, and the purpose of expanding beyond the core JS `console.log` is strictly to get better formatting capabilities. If you wonder why `file:` configuration is in `src/utils/logger.js` - it came with the usage sample. Feel free to delete it.


Finally, before we get to running live queries and mutations, a necessary walkthrough to see how to get *GraphiQL* to work - our Front End tool. A bit convoluted, but not too bad!