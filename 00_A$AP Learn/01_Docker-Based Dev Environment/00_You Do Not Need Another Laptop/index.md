### No More Extra Laptops, Dual Boots or VMs 

When it gets to learning new IT skills, the first problem to solve for a legacy developer often is acquiring or carving out a piece of hardware to install the required Operating System on before the real learning starts. Some can never get pass this problem, unfortunately for them.

The power of Docker makes this problem disappear: nothing is really needed other than Docker, and Docker runs on any OS - Windows, Mac, or Linux. So, use the equipment you already have!

Each software component of the system you learn will run in a separate Docker container, under the Operating System that the component requires, regardless of what OS your developer machine / laptop (the Docker *host*) uses.

The file system of your "host" will be used to store the data and program code accessible from the containers as well as from editors or IDE (Integrated Development Environment) you use normally on the host. 

The host will act as the *network* to bridge the containers via assigned ports. Via the host, processes running in the containers will also be accessible from, e.g., host's browser or even some internet-based software.

The *containerization* is not just a development convenience or a hack saving you from buying another laptop for this course. First and foremost, containerization is what runs *Production*. Using it in development becomes part of the natural lifecycle flow.

So, you can uninstall Vagrant and VM software (well, Windows Docker requires some VM components).

### MTR (Minimal Technical Requirements)

An average laptop with 4GB RAM and a few GB of spare disk space should be Ok.
<br>
In case you haven't worked with Docker much yet - we'll quickly review the few must-know things about it. 

But let's make sure your place of learning is equipped with the displays or accessibility hardware - as there will be screens upon screens to read, watch and interact with during this course.  