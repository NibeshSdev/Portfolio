---
layout: post
title: Docker in 6 minutes!
subheading: Docker
author: Nibesh
categories: Docker
# banner: https://bit.ly/32PAjtM
tags: Docker Container
sidebar: []
---

## Docker
Docker is a platform (or an engine) that runs the container. The software that hosts the container is called Docker Engine.

### Container
A conatiner is an isolated machine that runs on docker server. A container consists of everything it need to run i.e. source code, dependencies, runtimes, OS.

### Image
Containers are created from images. Images are the blueprint or the template for the container.

### Registry
Registry is a store for images. You can push/pull images from the registry and can use the same image to run many containers as you want.

![Docker flowchart](/assets/images/posts/04-2021/docker-flowchart.png "Docker flowchart")

### Run a simple Java application in a container

1. Create a java file. 
2. Add Dockerfile.
3. Build the image.
4. Run the container with image.

Step 1: Create a Java file. I named it **JavaDocker.java**.

```java
public class JavaDocker{
    
    public static void main(String[] args) {
        System.out.println("Testing Java with Docker");
    }  
}
```
Step 2 Create a file named **Dockerfile**.

```dockerfile
# Use an openjdk image with the SDK to compile java
FROM openjdk:8-jdk-alpine

# Set working directory
WORKDIR /out

# Get the source code inside the image 
COPY *.java .

# Compile source code
RUN javac JavaDocker.java

# Run the file
CMD ["java", "JavaDocker"]

```

Step 3: Build the image.

```terminal
docker build -t java-docker .
```
-t tag is optional and is used to give a name to the image. If name is not specified, docker will give a random name.
. is used to specify the path to be used as the build context

Step 4: Run the container with image.

```terminal
docker run java-docker
```

This application will output "Testing Java with Docker" in console and stops.

### Run Springboot application in a docker

Step 1: Create a spring boot application using [spring.io](https://start.spring.io/)

![SpringBoot Project Init](/assets/images/posts/04-2021/springboot-docker-project-init.PNG "SpringBoot Project Init")

Step 2: Create a REST endpoint

Create a HomeController.java file. Then, create a REST end point (or add following code).

```java
@RestController
public class HomeController {

    @GetMapping("/home")
    public String home(){
        return "Running spring boot in Docker";
    }
}

```

Step 3: Add Dockerfile

Create a file with named `Dockerfile` (without any extension). And add following code.

```dockerfile
# Use an openjdk image with the SDK to compile java
FROM openjdk:8-jdk-alpine

COPY target/*.jar springboot-docker.jar

ENTRYPOINT ["java","-jar","/springboot-docker.jar"]
```

Step 4: Add .dockerignore

Create a file with name `.dockerignore`. It is similar to gitignore in git. Add following codes so that docker ignores these files when copying files during build process.

```
.git
.mvn
.idea
```

Step 5: Build jar file using maven command

```terminal
mvn clean && mvn package
```

Step 6: Build image using docker command

```terminal
docker build -t springboot-docker .
```

Step 7: Run the conatiner with the image. We need to map the port from docker to outside using a `p` tag.
Here, we are mapping outside 8082 port to docker's 8080 port . 

```terminal
docker run -p 8082:8080 springboot-docker
``` 
Now check the [http://localhost:8082/home](http://localhost:8082/home) in browser. Did you notice that in the log, you can see the Tomcat started at 8080 port but our application is running on 8082 port. That is, our application is actually running inside docker on 8080 port but we cannot directly access the port inside the docker. Therefore, we map the docker port to oustide port that we can get access (here, 8082). 

When you try to stop the application (using Ctrl+C or else in the terminal), you see the application is still running. Because the docker/container is still running. You have to stop it using docker command. After stopping the container, you can remove the container if you want.

```docker
#list running containers
docker ps

#stop a running container
docker stop <container-id>

#delete a container
docker rm <container-id>
```

If you want to stop and remove the container using `Ctrl+C` or similar in the terminal then you can run the docker with following tags:

```docker
docker run --rm -it -p 8082:8080 springboot-docker
```
- -it switch allows to stop the container using Ctrl-C from the command-line
- -–rm switch makes sure that the container is deleted once it has stopped

### Push the image to DockerHub (registry)

Step 1: Create a [DockerHub account](https://hub.docker.com/) (if you don't have one).

Step 2: Tag the image with your Docker-ID as prefix `<docker-id>/<image-name>:<tag>`. `<tag>` is optional, but a good pratice.

```docker
docker tag springboot-docker <yourdockerid>/springboot-docker:1.0
```

Step 3: Login to your dockerhub from command line

```docker
docker login
# pass the credentials
```

Step 4: Push your image (docker push)

```docker
docker push <yourdockerid>/springboot-docker:1.0 
```

Then check your dockerhub. You can see the a new repository named `springboot-docker`.

### Bonus: Useful Docker Commands

```terminal
# help
docker run --help

# build image
docker build -t <image-name> .

# run a container
docker run <image-name>

# run detach container (run on background)
docker run –d <image-name>

# run container with port mapping
# or listening to incoming network
# -p outside-port:dockers-inside-port
docker run –p 8082:8080 <image-name>

# run a container. allows to stop and thenremove it with Ctrl-C in command-line 
# -it switch allows to stop the container using Ctrl-C from the command-line
# –rm switch makes sure that the container is deleted once it has stopped
# useful when testing, but in real scenerio, containers are long lived, so we don't use it in production
docker run --rm -it -p 8082:8080 <image-name>

#list running containers
docker ps

#list all running and stopped containers
docker  ps –a

#stop a running container
docker stop <container-id>

#delete a container
docker rm <container-id>

#delete all stopped container
docker container prune –f

#list all images
docker image ls

#download image regardless of present in your local
docker pull <image-name>

# remove image
docker rmi <image-id>
or
docker rmi <image-name>:<tag> (docker rmi server:latest)

#retrive logs of container
docker logs <container-id>

#or other options (-since, –from, –until, or –tail)
docker logs since 30s <container-id>

#inspect details of running container
docker inspect <container-id>


```




