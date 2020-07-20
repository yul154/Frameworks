1. grab this project from GitHub
2. Creating a Docker Image File
  * Using your IDE of choice, add a file and name it “Dockerfile” to the project root
  
3. Building an Image
  *  ran the build command to create an image from the Docker image 
  
4.  Running an Instance of our Image


build a Docker image
* create a docker file as a text file containing a recipe to build a docker image
* Use this docker file to construct a docker containe image by using docker build command
```
docker build -t app .  //Build an image from a Dockerfile

```

run image
```
docker run 

```

1. Create different ECR repository for different service in the console, the repository will centralized place for all my container images
2. Logging in to ECR in local machine,so i can upload my image
3. tag my docker images and upload it to repositories
4. In order to run th container and ECS container ,i am going to lunch that stack
5. i create a task definition for each of these repositories and turn those task definitions into a running service.




4. stack (prepared a CloudFormation template to set up a fresh BPC a cluster of docker host and a load balancer)

* . task definition(a list of configuration set for how to run my docke container ) from my application, i can launch this task definition as a service




The Docker image: An image is really a template fo creating the environmemnt you wanted

The Docker container 
* Containers allow a developer to package up an application with all of the parts it needs, such as libraries and other dependencies, 
* An instance of an image is called a container
*  if an image is a class, then a container is an instance of a class


The Docker registry is where you would host various types of images
