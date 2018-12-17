### Dot Environment Configuration File 

According to the modern methodology, runtime configuration must be passed to the NodeJS-based server via Operating System's Environment Variables. No more files with parameters and encoded passwords in them. Leave those with the legacy systems they were invented for. [The Twelve-Factor App](https://12factor.net/config) explains the background of this approach.

Containerization technologies, both at the base Docker level and at the core of orchestration systems, provide simple mechanisms of setting up the OS Environment when creating a container. Controversially, the values come from configuration files - although, stored in the orchestration system vs. in the app's directory.

Conveniently, Docker loads configurations from simple `name=value`, new-line delimited text files, which is the same format as the one used by NodeJS package [dotenv](https://github.com/motdotla/dotenv). So, the same configuration file can be used to feed either Docker container OS Environment setup or NodeJS runtime directly via the `dotenv`. 

NodeJS handles access to the OS Environment variables via the `process.env` object. `dotenv`, upon the start of the server process, looks for the file named `.env` in the root directory of the project and, if the file is present, loads data from it into `process.env`. By design, `dotenv` does not override values already present in `process.env` - those that come from the OS Environment. So, the OS Environment remains the master, `.env` file is used to complement but not supersede the configuration passed to the NodeJS server from the OS Environment.

As `.env` contains machine-specific configuration, it should be excluded from the Git project - it is listed in the `.gitignore`. File `.default.env` is the *template* to create `.env` from, and it is in the Git project. Values in `.default.env` are those that a typical local dev environment would use. Copying `.default.env` into `.env` and reviewing its content will be a step in the environment setup later in the hands-on part of the course.

So, as far as passing configuration to NodeJS per the 12-Factor approach - the production  deployment into the orchestration does it (we talk about this more in the last chapter). In development, we do use the configuration file `.env`.

Worth noting that the 12-Factor paradigm really shines when the configuration is driven by service *discovery*, which is a key feature of orchestration frameworks. As the container is created, the framework passes the necessary details into its OS Environment, some of which can be pre-set but others would come dynamically from the content of other deployed components. E.g., if we use a database service external to the orchestration deployment, we have to explicitly provide the host name and admin credentials to the orchestration layer to be passed to certain containers. But for services that are started by the orchestration itself - we can let the orchestration layer generate keys, names, passwords and distribute those to other containers automatically, without the human admin managing any of that. 


## .env File Use For docker-compose

By default, `docker-compose` will read `.env` present in the directory it runs from to get values for Environment variables which are not already set in the OS Environment of the host - very similar to what `dotenv` does, so the pattern is consistent across various technologies


Without a pause, jumping into JS code review next
