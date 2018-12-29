### AWS ECS

If you read about Cloud and containerization trends you'd likely pick Kubernetes (K8s) as the platform to run your containers on. Next, you start thinking of *installing* it so you can get *hands-on* with it. You'll realize very soon that installing K8s is not trivial. So, next, you look for a K8s *service* to use. And, you'll get even more confused and lost - because if you don't know what K8s is and you can't install it - how can you choose a service right for your use case and figure out how to actually *utilize* it running your apps production? So, you hire consultants. "Show me where North is - and I'll tell you exactly what direction is South". 

Well, you have to figure the container deployment stuff out, the sooner, the better. 

Before getting deep into K8s maze, you may decide to try [AWS ECS](https://aws.amazon.com/ecs/) out, running it in the EC2 mode. Especially, if you have some experience setting up software in AWS EC2. 

The details are out of scope of this course, but this is what you'll set up, production-ready:

- Load Balancer with HTTPS termination
- Auto-scalable Container Orchestration ECS Cluster
- Auto-scalable Service representing your GraphQL Server, with the configuration, e.g., the JWT Public Key and MongoDB password securely recorded in the ECS. Each instance of the server running in a container started from the image built in the project directory using the Dockerfile: 

  ```
  FROM node:8
  WORKDIR /app
  COPY . /app/
  RUN npm install
  RUN npm run build
  CMD npm run start:production
  EXPOSE 8080
  ```

- Code Pipeline to automatically re-deploy the service when code updates are pushed into the GitHub project repository 
- Server log collection and analysis

At a fraction of cost and less complexity than a full-scale Kubernetes service, ECS provides an automated production ready platform for deploying your GraphQL server and auto-scaling it to the required load.

One thing to realize is that depending on the overall design of your application portfolio you may *never* need to *sophisticate up* to full-scale Kubernetes. If you keep the container-based set of applications clean and simple - you don't need all the bells and whistles that help running containerized stack as if it was a bunch of old-style VMs or server clusters. Simplify the base app design and utilize serverless, and you may soon not recognize your portfolio - in a good way.

Whatever you do - do not try installing anything or running yourself. Leave the platform stuff to the professionals to handle. Focus your effort on deciphering their services documentation and offerings: not so easy.
<br>
Last thing to cover before we run Production - automated testing. Let's see what that is about