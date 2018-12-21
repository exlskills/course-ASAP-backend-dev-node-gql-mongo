### Winston Logger

NodeJS doesn't have a built-in logging framework. The standard beloved JS `console.log` can be used, but there are several (competing) 3rd party solutions that work pretty well.

The demo project uses the `winston` package. As it is required for a package to have a name, `winston` can be just as good a name for a logger as, e.g., `log4j`

Traditionally, server log configuration has been an important step in the install process. Logs would be rotated, archived, passed on to a monitoring solution. `winston` has a complete set of capabilities configuring log outputs. 

However, containerization may be the death for old-stile loggers. Basically, the console log output is all we need when running a server in a container: for either dev or production. As Docker reads the console-out, it seamlessly passes it on to the monitors and/or consolidators, bypassing anything that has to do with files and rotations.

So, in the demo project, the use of `winston` is limited to configuring the console output, and the purpose expanding beyond JS `console.log` is strictly to get better formatting capabilities. Although, you'll see `file:` configuration in `src/utils/logger.js` - as it came in with the sample. Can definitely be deleted.


Finally, before we get to running live queries and mutations, a necessary walkthrough how to get the GraphiQL - our Front End - tool working. A bit convoluted, but not too bad!