# Docker Overview

This will go through setting up a simple flask app, running it in a Docker container, and saving the image.  Key concepts we'll cover:

* Docker images
* Docker containers



## Images

First look at the images we have -- show how there's nothing.

```console
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
```

### docker pull 

First step is to use "docker pull" to grab a base image.  Le't use "ubuntu:latest"

```
$ docker pull ubuntu:latest
Pulling repository ubuntu
c4ff7513909d: Download complete 
511136ea3c5a: Download complete 
1c9383292a8f: Download complete 
9942dd43ff21: Download complete 
d92c3c92fa73: Download complete 
0ea0d582fd90: Download complete 
cc58e55aa5a5: Download complete 
```



```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu              latest              c4ff7513909d        15 hours ago        225.4 MB
```


## Containers

### Start an interactive container

### Running commands in a bash shell

### Committing changes

### Opening a port

### Changing the command to start a program automatically




## Cleaning up

This is a great post about managing the mess that docker can leave behind

[Remove Untagged Images From Docker](http://jimhoskins.com/2013/07/27/remove-untagged-docker-images.html)

This will kill all running containers:

```
$ docker rm $(docker ps -a -q)
```

You can then delete all the random containers using his other example, which I can't get to work for some reason...










