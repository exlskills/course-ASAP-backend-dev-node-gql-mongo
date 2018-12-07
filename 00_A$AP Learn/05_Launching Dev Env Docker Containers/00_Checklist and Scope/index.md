### Checklist and Scope

Earlier in the "Docker-Based Dev Environment" chapter we went over installation requirements. Even though we minimize what gets installed on the dev box, any installation is always a chore and a potential bog down point. If you still can't get these components loaded, give us a ping. Although, if your hardware is really old and weak - you've got to get it upgraded.

As the recap, this is what you need to have installed to continue:

- [ ] Docker Community Edition Engine, Docker Compose
  * [ ] On Windows, enable Docker Shared Drive in `Settings`
- [ ] Terminal Shell
- [ ] Git
- [ ] VSC
- [ ] MongoDB Compass

## Have The Demo Project Cloned Locally

At this point, the demo project code should be cloned into a local folder on your box that will be shared with the NodeJS Docker container. You can fork the [EXLskills GitHub repository](https://github.com/exlskills/demo-gql-mongo) into your GitHub account or clone directly from the EXLskills repo. 

### Containers We Run

- MongoDB
  * we'll launch a MongoDB container to host the dev database
  * if you already have an instance of MongoDB running - you can use it for the course and skip launching another container. However, you'll need to adjust the MongoDB URL used by the NodeJS server to connect to your instance. Hopefully, if you have MongoDB running - you'd be an expert on how to do that already
  * we will not force the location of the MongoDB data volume on the host - we'll allow Docker use its default. We want to preserve the data on container restarts, but we won't work with this data via the host's file system
- Linux NodeJS
  * we'll launch a NodeJS container to run our application from its shell, via the terminal
  * we'll share the project folder on the dev box (*host*) with this container 

### Docker Compose or Not

Docker CLI is one of the core skills to absolutely have. Basically, Docker Compose is just a convenient way managing multiple containers via one YAML file vs. individual CLI. As we only have two containers in scope - let's just CLI them for now.

Let's get to work!