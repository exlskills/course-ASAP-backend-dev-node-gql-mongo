### Dot Environment Configuration File 

All runtime configuration must be passed to the NodeJS-based server via Operating System's Environment Variables. No more settings files with a long list of parameters and encoded user names and passwords. Those are left with the legacy systems they were used in. [The Twelve-Factor App](https://12factor.net/config) has the background of this approach explained.

As the container represents a running Linux process, we can configure the Environment of that process and set everything that the NodeJS application running in the container needs. Containerization technologies, both at the base Docker level and at the core of orchestration systems, provide simple mechanisms of controlling the container's OS Environment. Well, in the end there are still usually configuration files present, but at the level of container orchestration vs. the inside the project directory.

Conveniently, Docker loads configuration from simple `name=value` new line delimited text files, and JS has a package called [dotenv](https://github.com/motdotla/dotenv) enabling the use of these files for configuration as well. 

NodeJS handles access to the OS Environment variables via `process.env` object, and `dotenv` reads data from the file named `.env`, if such a file is present in the root directory of the project. By design, `dotenv` only sets values that are not already present in the OS Environment, so it complements but does not override the OS Environment.

As `.env` by definition of its purpose contains specific configuration used at runtime by the actual system, it should never be exchanged across dev instances via Git - so it is listed for exclusion in `.gitignore`. Instead, the project contains `.default.env` file, which should be copied into `.env` on the dev workstation and then the `.env` edited as needed. Realistically, `.default.env` is the `.env` file for a typical dev workstation, pointing to the local resources and listing default names and test passwords. 

So, even though we like, use and support the 12-Factor configuration paradigm, it really shines when applied in the production deployment, where the configuration is sourced from the orchestration environment's service discovery and gets passed into the individual containers' OS Environment. At the dev stage, we maintain a stencil `name=value` configuration file that serves multiple purposes: it lists the configuration variables in scope of the project, contains common dev defaults, can be copied and used to run the NodeJS process via the command line or to control Docker container's OS Environment. Note, that Docker sets the OS Environment of the container once - when the container is created.

## .env File Use For docker-compose

By default, `docker-compose` will read `.env` present in the directory it runs from to get values for Environment variables which are not already set in the OS Environment 
