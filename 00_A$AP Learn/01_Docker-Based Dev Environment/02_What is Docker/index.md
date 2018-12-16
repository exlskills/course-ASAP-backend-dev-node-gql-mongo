### 2-minute Docker Briefing 

Docker is a continually running (daemon) program. It runs on the *host* machine and manages components called *images* and *containers*. Docker containers represent machines with their own OS and other software and resources - pretty much similar to traditional *Virtual Machines*, except that they run within the host's OS paradigm, whereas VMs are completely isolated from the underlying host's OS by the *hypervisor*.

From the software developer's point of view, Docker containers are no different than VMs. Both are *virtual* (vs. *bare metal*) and run OS different from that of the host. To a hardware-focused person, though, containers are very different from traditional VMs because VMs use independent *slices* of the available host resources, whereas containers are ultimately *programs* running within the host's OS and *managed* by Docker.

This is the last time we're going to compare containers and VMs. Let's go by the developer's view and think of containers as independent machines with convenient resource sharing options with the host machine. In other words, when you have a laptop with two Docker containers running on it - you've got *three machines* running - great!

Containers are usually thought of as *running* (machines). However, they can be *not running* (exited) or *starting/stopping* - similarly to your laptop being in suspended mode or going in/out of it.

Docker components are managed and controlled via Docker Command Line Interface (CLI). 

As containers run within the host, there are obvious security risks of exposing host's or other containers' stuff to malicious containers - always be mindful of that.

Containers are created from images. Docker images are like VM images - they have the Operating System and programs installed, and usually some additional files with configuration and data present. So, in a way, images are like not running containers, but in the Docker universe that's not a correct understanding. Images have a strong focus on portability - can be moved around - and purpose of producing good containers. They (usually) have all the information for the container to start and run successfully. Containers are supposed to be ephemeral - they can't be moved or reused. They run something and then they eventually disappear when the goal is achieved, and no one ever remembers they existed.

In practical reality, though, containers can be used in any way useful and convenient - run, stopped, restarted. Just keep in mind that the container's file system by itself is *not persisted*, and once the container is "removed" - it is gone. If you do want to keep updates to the file system done by the container - you must explicitly define volumes shared between the container and the host when you start (`run`) the container for the *first* time.

In accordance with the theory, containers must always run some process when they are started. The default process to run is defined in the image, and an override can be put into the `docker run` command. If nothing explicit occupies the container's runtime - the container stops (exists). This is annoying when you just want to start a generic container that runs as a daemon machine waiting for you to access it and try some things out, but there are easy ways to emulate a dummy daemon to keep the container from shutting down.

### The Three Docker Commands 

Here they are: `docker build`, `docker run`, `docker exec`.

How Docker images are created? As "chicken and eggs", a container can be *saved* as an image, but the proper way of creating images is via `Dockerfile`. Per the Infrastructure-as-a-Code rules, any and all installation and configuration steps for anything must be written down and packaged - and `Dockerfile` is a perfect place for that. `Dockerfile` is literally a sequence of commands that configure the image - install all the software, etc. Those commands are executed via the `docker build`. 

In the `Dockerfile`, you must specify the source image (can be multiple - separate for different build steps). `docker build` runs on the host and, technically, is identical to `docker run` - both start a little machine (container) and execute commands in it, but, somewhat unfortunately, `build` and `run` have their own, not always overlapping options, and sometimes act differently in similar situations. If the `build` errors out, there is no clean way to debug and troubleshoot it, although you can usually see the stopped container that you `commit` as an image and try starting for manual troubleshooting.

So, `docker build` is a complex process (lots fo room for improvement), but the other two commands are very simple: `docker run` creates and kicks of a container from the built image, and `docker exec` executes a one-off command in the running container. `docker exec` is often used to open container's console shell, e.g., `bash`, and start poking around.

The commands have a long (hugely long) list of parameters that control host-container coordination: shared volumes, use of networking and ports. Alternatively, `docker-compose` is designed to manage the parameters in a YAML file, for a group of containers vs. one at a time - different flavours of the same tech foundation.

In the demo project, we do not *build* any images, just use standard ones, so there will be no `docker build`. But the other two commands will be used quite a lot.


Containers are cool, but their real value shines when it gets to *container orchestration* - let's look at that next!
