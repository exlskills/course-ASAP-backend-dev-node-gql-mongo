### Launching Dev Environment Docker Containers on Windows

**Please read the Launching on Linux section if your laptop OS is not Windows**


### MongoDB

Make sure that port 27017 on your box is not already used by another local instance of MongoDB. You can change the left-side (host side) port number in the `docker run` command below to use a different port instead. If the port you are trying to use is already engaged - you'll get an error message when launching the container. If you do, remove the container (`docker rm mongo`), troubleshoot the port problem and launch the container again.

In the PowerShell, from any folder (applies to all commands in this section), run

```
docker run -d --name mongo -p 27017:27017 -v mongodata:/data/db --restart unless-stopped mongo:3.6
```

The command options used are:

- `-d` - run the container in the daemon mode, meaning that it keeps on running for as long as we need it
- `--name mongo` - the container name we can refer to it by in other docker commands
- `-p 27017:27017` - host port 27017 is mapped to the container port where MongoDB is listening on (27017, as per mongo official image docs)
- `-v mongodata:/data/db` - map `mongodata` *docker volume* on the host to the `/data/db` folder inside the container. Docker volume is automatically created if it doesn't exist. `/data/db` is the documented folder where containers launched from the official mongo image keep their database files. 
 Note that the container's startup process is designed to reuse existing database files in the folder. This is a typical feature that allows seamlessly restarting or relaunching containers over the same database content. If the folder is empty, the database is initialized with the documented defaults. The downside of *reusing* is that if the files are corrupted or wrong, the container may fail. In that case, cleanup or removal of the docker volume resolves the issue
- `--restart unless-stopped` - restart the container if it is not running, unless it was explicitly stopped via the `docker stop` command. We want the running container auto-restarted on system's reboot or docker daemon restart
- `mongo:3.6` - the name of the image for the container: [official MongoDB version 3.6 Docker image](https://hub.docker.com/r/library/mongo/tags/3.6/)

Note that we do not provide any *command override* to the container that would go after the image name at the end of the `docker run` statement. In this case, the default startup process defined in the image is launched. 

If you haven't used `mongo:3.6` image on the box before, docker will pull it from dockerhub. You'll see this message

```
Unable to find image 'mongo:3.6' locally
3.6: Pulling from library/mongo
```

Once the container is started, Docker prints the long container ID and returns back to the PowerShell prompt. The absence of any other messages would indicate that the container was likely launched successfully.

At this point, `docker ps --all` should show the `mongo` container running. `CREATED` and `STATUS` should be off by a second or so. Repeat `docker ps` a few times to ensure that the container keeps running - the `STATUS` seconds count should steadily grow as you're retrying `docker ps`. What you're looking for is to make sure that the container is not restarting due to some errors. With `--restart unless-stopped` set, if the process inside the container errors out *after* the container initially starts up - Docker daemon will keep restarting it. Note that if the port is occupied or Docker encounters some other configuration issues - the container won't *start* at all, and you'd get a message in the console. Restarts don't generate console messages, so you should monitor `docker ps` and then check the container's log: `docker logs mongo`. Ignore the MongoDB performance warnings in the log - this is just a dev db. The log is your source for troubleshooting if the container keeps restarting.

`docker volume ls` outputs a bunch of lines, including one with `mongodata` - the name we used on the left side of the `-v` parameter.

`docker volume inspect mongodata` shows the folder where docker created the drive, e.g., `"Mountpoint": "/var/lib/docker/volumes/mongodata/_data"`. Wait a minute, where is `/var/lib/docker/volumes/mongodata/_data` in my Windows File System? It's hidden in the footprint of the buffer Unix VM that Docker Launches on Windows to trick itself that it is running on Unix. Never mind, as we said, you would not be accessing this folder from the host anyway.

`docker inspect mongo` shows details of the running container.

On failures or when you want to clear everything out, `docker stop mongo` and `docker rm mongo` remove the container. You can view the log of a *stopped* container, but the log is gone once the container is *removed*.

The shared volume will be automatically reused when you relaunch the `docker run` command for the container. The data on the volume is supposed to be preserved when you stop or remove the container. You can remove the volume via `docker volume rm mongodata` if you want to start fresh or wrap up.

### NodeJS

Let's assume you cloned the demo project repository into `C:/docker-vol/demo-gql-mongo` on your Windows box. Drive `C` is marked as shared in your Windows Docker Settings. Then your container launch command should be as follows:

```
docker run -d --name node-dev -p 8080:80 -v C:/docker-vol/demo-gql-mongo:/myapp --restart unless-stopped node:8 tail -f /dev/null
```

We're exposing container's port `80` on host port `8080`.

Note that `-v C:/docker-vol/demo-gql-mongo:/myapp` contains a path on the left (host's) side - this indicates sharing a specific *folder* vs. using a Docker *volume* as we did for the `mongo` container. Alternatively, a Docker volume can be created ahead of time, linked explicitly to the folder that we need to share. 

On Windows, the container may lose visibility of the shared drive's content on OS reboots (and, there is no reliable way to prevent Windows from self-rebooting, other than switching to Linux). The Docker would restart the container per `--restart unless-stopped` but when you get inside, `/myapp` would show empty. If this happens, a manual `docker restart node-dev` should fix the problem.

`tail -f /dev/null` is a dummy daemon command that puts the container into an infinite waiting loop. Run `docker stats` command to see the resource utilization by the containers. `node-dev` takes up a tiny memory slice and zero CPU when running in this "dummy" daemon mode. 

Notice that the `LIMIT` of memory showing in `docker stats` may be lower than the total available RAM capacity of your box. Infamously, Docker on Windows sets up some low defaults that may be insufficient for your Docker-based Devstack if you have lots of containers running. Keep that in mind and check whether the limit can be increased if you run into this problem - out of scope of this course.

Now, let's get into `node-dev` shell. From the PowerShell, run:

```
docker exec -it node-dev bash
```

A command prompt like `root@<container ID>:/#` will indicate that we're inside the container's shell. Note, we are the `root` in the container. Run

```
cd /myapp
ls -la
```

`/myapp` is the folder inside the container that is mapped to the host's folder with the demo project files, per `-v C:/docker-vol/demo-gql-mongo:/myapp`. `ls -la` should list the project files and directories. Note that they are owned by `root` and belong to the `root` group, as far as the `node-dev` concerns.

Note that if we build our own image from `node:8`, we can set `/myapp` as the default entry folder and save ourselves effort switching to it every time we re-enter the shell. A homework for you.

If you're curious what Operating System you're working in, run `cat /etc/*-release`.

`exit` ends your terminal bash session and puts you back into the PowerShell.

What is `-it` in the `docker exec` (may also be seen in `docker run`)? One answer can be that `-it` means that we're using the IT Professional mode. In practical terms, `-it` is the opposite of the `-d` *daemon* mode of the container: you get both `-i` *input* open and `-t` for *tty*, so you can use the container's terminal. You can either launch a container in the `-it` mode and start `bash` form the get go, or you can `exec` `bash` in the `-it` mode on a container launched in the `-d` mode. Note that some Unix containers are built from images that do not have `bash` enabled, so you'd need to try other shells.
<br>
Now that we have the dev containers running, we are finally ready for the action!
