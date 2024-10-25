---
title: "Shipping Docker - Learn to build Docker Image for ubuntu 20.04"
template: "post"
category: "Docker"
tags:
  - "docker"
  - "ubuntu"
  - "javascript"
published: true
draft: false
date: "2021-07-14T23:46:37.121Z"
slug: "/posts/shipDocker"
description: "In this article you'll learn how to build a Ubuntu 20.04 image. But before starting to build the image, let's quickly have a brief about docker."
---

### Why Docker??

<figure class="float-center" style="width: 1000px">
	<img src="/media/docker.jpg" alt="docker">
</figure>

Docker helps you to `build, ship and run your app, anywhere`. It helps you to pack all you libaries, binaries, OS and application related services readily available to build your app. Hence termed as containerized technology easy for people to use.

Docker is like a virtual machine, but unlike a virtual machine. Instead of creating a whole virtual operating system, this application allows you to use the same kernel as of the system the docker is running on.

### What is Docker image?

Images are files used to execute a set of code within a docker container. They contain a set of commands to build a docker container. An image can be compared as a snapshot of a virtual machine. The image itself contains commands needed to build a container using the libraries, dependencies, tools, environment files, configs inorder to run the application within a container.
Containers are dependent on images, because the images act as a starting point, which helps to build a container (runtime environment).

Here’s is a list of docker commands

- `docker-compose up -d` - Builds, (re)creates, starts, and attaches to containers for a service.
- `docker ps` - list the containers
- `Docker images` :- lists all the images
- `docker run` - Runs a command in a new container.
- `docker start` - Starts one or more stopped containers.
- `docker stop` - Stops one or more running containers.
- `docker build` - Builds an image from the Docker file.
- `docker pull` - Pulls an image or repository from a registry.
- `docker push` - Pushes and image or repository to a registry.
- `docker exec` - Runs a command in a run-time container.
- `docker search` - Searches the Docker Hub for images.
- `docker commit` - Create a new image from a container’s changes.

To know the full list of commands, do refer the [Docker documentation](https://docs.docker.com/engine/reference/commandline/docker/)

### Installing Docker

Before we start, we need to have Docker installed, whether it's on your local machine or a remote server. Refer to the [official site](https://docs.docker.com/get-docker/) to install and setup docker on your machine.

Once you install the Docker, you will be able to view & access the GUI interface, which would look like:

<figure class="float-center" style="width: 1000px">
	<img src="/media/dockerGUI.png" alt="dockerGUI">
</figure>

### Pull a docker image of Ubuntu machine with version 20.04

<br> 1. Inorder to use Ubuntu 20.04, we need to pull the image from dockerHub. Docker Hub is a hosted repository service provided by Docker for finding and sharing container images with anyone over internet. For more information refer to this [site](https://www.docker.com/products/docker-hub).
<br/>

To pull the image, run the below command:

```
$ docker pull ubuntu:20.04
```

This will pull the Ubuntu image and will be able to see it in the listing screen within the Docker GUI interface. Also you can verify the pull, via CLI command

```
$ docker image ls -a

REPOSITORY                       TAG       IMAGE ID       CREATED        SIZE
ubuntu                           20.04     7e0aa2d69a15   2 months ago   72.7MB
```

<br> 2. Let's run the Ubuntu 20.04 image using the command:

```
$ docker run -it --entrypoint "/bin/bash" ubuntu:20.04
```

This will build and run a container for Ubuntu 20.04 image. Also we have attached a entry point, that will launch a bash shell. <br/>

### Build a Ubuntu 20.04 container using a docker compose

Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration.

Here we will build the ubuntu 20.04 container using docker compose.

1. Create `docker-compose.yaml` file and add necessary configurations to pull and build the container. The file looks like this:

```
version: '3.9' # optional since v1.27.0
services:
  ubuntu:
    image: ubuntu:20.04
    tty: true
```

2. List the docker containers using command:

```
$ docker ps -a
CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS                        PORTS     NAMES
e547b463ed0e   ubuntu:20.04   "/bin/bash"   4 weeks ago      Exited (0) 34 minutes ago               testProject_ubuntu_1
```

3. Run and build the `docker-compose.yaml`

```
$ docker-compose up -d
Docker Compose is now in the Docker CLI, try `docker compose up`

Starting testProject_ubuntu_1 ... done
```

4. Now you can access the container, simply by executing:

```
$ docker exec -i -t testProject_ubuntu_1 /bin/bash
root@e547b463ed0e:/#

```

There you go, we have created a Docker container of Ubuntu 20.04. That's all 😄
