### Inter-Container Communications

Containers may be given access to host's OS-managed ports. This enables the containers and the host communicate and access each other's services as well as expose containers' ports to external systems. E.g., a container that runs a database can be started with its port 27017 mapped to the host's port 12345; then another container running a web server would be able to access the database via the host's port 12345. 

Technically, Docker starts and maintains its own *network* on the host - in the private address space `172.17`. Each container gets an IP address, and the host's address is 172.17.0.1. In the above example, the web server would access the database at `172.17.0.1:12345`. In addition to the default network `172.17`, other docker networks can be configured and used on a given host.

The ultimate usability of *containerization* comes from assembling multiple hosts into clusters - container *orchestration*. From the developer's standpoint, multi-host configurations are fully transparent: the orchestration engine is responsible for managing the network across all hosts, and everything seems to work exactly like it does on a single host, yet the multi-host platform creates unlimited scalability to run lots of containers. In reality, though, the orchestration adds a lot of complexity to managing container runtimes. 

Everyone is familiar with the cloud's concept of adding more VMs when the application requires more computing power or replacing broken VMs on the fly with new ones. With containers, the level of abstraction shifts from VM to container. As the container orchestration engine can manage the process of adding/removing containers based on the app workload and replacing misbehaving ones - we don't have to worry about cloud VMs anymore.  

Of course, even though the hosts are logically virtualized, somewhere down the virtualization path they are placed on physical hardware. So hardware-related issues propagate into container orchestration no matter what:

- containers' own file system is ephemeral and disappears when containers are replaced 
- volumes shared with hosts are mapped to some physical storage devices that often cannot be moved around at will, so replacement containers must be put somewhere where they get access to that storage
- *network storage* adds latency and complexity 
- containers running on a given host compete for the finite resources of the physical machine the host runs on 
- distributed clusters are great for redundancy and availability but add latency to cross-container communications

Solutions to these issues may undermine advantages of containerization or drive up costs. When containerization becomes more expensive than pure VM-based cloud - people either revert back to VMs or keep exploring options, e.g., go *serverless* 

### Proper Solution Design for Containerization Success

Just like with moving monster legacy server-based architecture to VMs, the success of VM to container move is rooted in proper app design. If all we do is through an existing VM app into containers - we may be doing disservice to ourselves and our customers.

Examples of how the traditional VM-based design makes the app a bad fit for containerization are plentiful. To mention just a couple:

- a dedicated component for in-memory caching that used to run on the same VM as its consumers becomes a performance killer when placed in a container and deployed into a remote part of the cluster. Popular engines like Kubernetes use the concept of Pods to "assemble" several containers together before placing on a host - well, larger components require larger hosts, which in their extreme become bigger, less efficient and harder to configure than VMs 
- an app that requires lots of CPU and memory at startup compare to the runtime average would need to be placed on a host with that much capacity always available or risk a serious overload at the worst moment. Orchestration does allow configurable *overbooking* of capacity assuming that host tenant containers would not all spike up to their assigned limits at the same time. Unfortunately, in the scenario when the demand is high and the engine decides to add more containers to support it, heavy load startup of those new containers strain the system even further and may cause a complete freeze and blackout, effectively killing the entire orchestration cluster. So, frameworks like Django that by default run lots of checking at server startup may become a problem 

### Basic Principles of App Design for Container Orchestration 

- Choose light and simple base apps frameworks with low startup and runtime capacity requirements 
- Avoid using persistent states, "sticky sessions", container-level persistent datasets. Design to enable seamless and transparent container placement and redeployment natively by the orchestration engine 
- Use services outside of the orchestration engine's management scope for inherently persisted systems such as databases or queues 
- Design effective in-memory caching solutions balancing cross-container networking latency and single placed component's footprint

<br>
Now that we've got the overall scope of Docker (and even some Kubernetes) covered, let's install Docker on the laptop to get it ready for the demo project work, but, first, a quick check to make sure the CLI terminal / console / shell is up to par for the work coming
