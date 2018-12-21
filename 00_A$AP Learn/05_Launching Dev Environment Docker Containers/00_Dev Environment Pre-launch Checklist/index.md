### Dev Environment Software Requirements Checklist

In the "Docker-Based Dev Environment" chapter we went over the standard software installation process for the components necessary to support our efficient Dev Environment. Any installation, no matter how minor, can be a chore and a bog down point. If you still haven't had success setting any of those components up, give us a ping. 

As a recap, this is what you need to have installed to continue from this point in the course on:

- [ ] Docker Community Edition Engine
  * [ ] On Windows, enable Docker Shared Drive in `Settings`
  * [ ] On Unix, enable non-root Docker access
- [ ] Terminal / Console Shell
- [ ] Git
- [ ] VSC
- [ ] MongoDB Compass


## Next, Have The Demo Project Cloned Locally

Till this point, we've been *reviewing* the demo project code, and we could do it perfectly fine browsing directly throw the content of the [EXLskills GitHub repository](https://github.com/exlskills/demo-gql-mongo) via the [EXLcode Chrome extension](https://chrome.google.com/webstore/detail/exlcode-vs-code-based-onl/elcfpiphmolcddmecegalaikjiclhdjc?hl=en). Now, to execute the code, we need to clone the repository locally.

The demo project code will be accessed from both the host (your laptop) and the NodeJS Docker container that we're about to launch. Let's pick a good spot and clone the code into there:

- On Windows, the folder needs to be set up on a Drive listed in *Shared Drives* in Docker->Settings. Assuming that `C:` drive is shared with Docker, the project can be cloned into `C:/docker-vol/demo-gql-mongo`
  * Open PowerShell and create the top level directory `mkdir C:/docker-vol`. Note, PowerShell understands the *normal* slash `/`. What a step *forward* for Windows! 
  * Change directory `cd C:/docker-vol`
  * Clone the demo project repository

  ```
  git clone --depth=1 https://github.com/exlskills/demo-gql-mongo.git
  ```

  `--depth=1` is used to shake off the *commits history* that is not relevant for the purpose of working with the demo project

  If you *forked* the project, you can clone the fork instead:

  ```
  git clone --depth=1 https://github.com/<your org>/<your fork>.git C:/docker-vol/demo-gql-mongo
  ```

- On Linux, you can place the folder into the user *home* directory, e.g., `~/docker-vol/demo-gql-mongo`
  * In the shell, create the new directory `mkdir ~/docker-vol` 
  * Change directory `cd ~/docker-vol`
  * Clone the demo project repository

  ```
  git clone --depth=1 https://github.com/exlskills/demo-gql-mongo.git
  ```

  `--depth=1` is used to shake off the *commits history* that is not relevant for the purpose of working with the demo project

  If you *forked* the project, you can clone the fork instead:

  ```
  git clone --depth=1 https://github.com/<your org>/<your fork>.git ~/docker-vol/demo-gql-mongo
  ```


### Docker Containers We Will Run

- MongoDB
  * we'll launch a MongoDB container to host the dev database
  * if you already have an instance of MongoDB running - you can use it for the course and skip launching another container. However, you'll need to adjust the MongoDB URI used by the NodeJS server to connect to that instance. If you run MongoDB - you're an expert on how to set URI to get to it from applications
  * we will not force the location of the MongoDB data volume on the host - we'll allow Docker use its default. We want to preserve the data on container restarts, but we won't be working with the MongoDB raw storage via the file system, so we don't care where it is
- NodeJS
  * we'll launch a NodeJS container to install the project and run our application from the container's shell
  * we'll share the project's folder cloned on the dev box (*host*) with this container 


### Docker Compose or Not

Docker CLI is one of the core skills every developer must absolutely have. Basically, Docker Compose is just a convenient way to manage multiple containers using one YAML file vs. typing individual CLI for each. As we only have two containers in scope - let's just CLI them for now.


Let's get to work! The following two lessons are parallel ones for Windows and Linux - *skip* the one that does *not* apply to your host OS