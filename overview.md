# Docker Overview

This will go through setting up a Docker image that will run a simple flask app.  The goal is to provide a a "Hello, world"-style introduction to the key concepts in Docker, such as images, containers, commits, tags, layers, and so forth.  We'll walk through:

* An overview of the Docker commands
* Pulling down a base image
* Running a container based on an interactive shell
* Install our app and its dependencies in the shell
* Committing the container at each major checkpoint
* Running our container in daemon mode
* Viewing our app in the browser

The key concepts to remember as you walk through this example include:

* an *image* is a specific state of a filesystem
* an image is composed of *layers* representing changes in the filesystem at various points in time; layers are a bit like the commit history of a git repository
* a *container* is a running process that is started based on an image
* you can change the state of the filesystem on a container and commit it to create a new image
* changes in memory / state are not committed -- only changes on the filesystem [VERIFY THIS!]

## Docker command overview

Like boot2docker, docker uses the git-style command format:

```
$ docker [OPTIONS] COMMAND [arg...]
```

The following table, taken from the "docker help" command, provides a quick summary of the commands.  As you scan the list (which I've left in alphabetical order), you'll notice 3 key groups of commands:

* Commands related to managing images, such as *images*, *build*, *save*, *rmi*, and *tag*.
* Commands related to containers, such as *run*, *ps*, *kill*, *restart*, *top*, and *pause*.
* Commands related to the [Docker hub](https://hub.docker.com/), such as *login*, *search*, *pull*, and *push*. (More on this in the next chapter.) 



| Command  |  Description
|----------|-----------------------------------------------------------------------|
| attach   | Attach to a running container
| build    | Build an image from a Dockerfile
| commit   | Create a new image from a container's changes
| cp       | Copy files/folders from a container's filesystem to the host path
| diff     | Inspect changes on a container's filesystem
| events   | Get real time events from the server
| export   | Stream the contents of a container as a tar archive
| history  | Show the history of an image
| images   | List images
| import   | Create a new filesystem image from the contents of a tarball
| info     | Display system-wide information
| inspect  | Return low-level information on a container
| kill     | Kill a running container
| load     | Load an image from a tar archive
| login    | Register or log in to the Docker registry server
| logs     | Fetch the logs of a container
| port     | Lookup the public-facing port that is NAT-ed to PRIVATE_PORT
| pause    | Pause all processes within a container
| ps       | List containers
| pull     | Pull an image or a repository from a Docker registry server
| push     | Push an image or a repository to a Docker registry server
| restart  | Restart a running container
| rm       | Remove one or more containers
| rmi      | Remove one or more images
| run      | Run a command in a new container
| save     | Save an image to a tar archive
| search   | Search for an image on the Docker Hub
| start    | Start a stopped container
| stop     | Stop a running container
| tag      | Tag an image into a repository
| top      | Lookup the running processes of a container
| unpause  | Unpause a paused container
| version  | Show the Docker version information
| wait     | Block until a container stops, then print its exit code



## Working with Images

As a first step, let's use "docker pull" to grab the latest release of Ubuntu: 

```console
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

When you pull an image you'll see each dependent layer being downloaded, as well.  Once all the layers are finished downloading, you can run *docker images* to see information about what you have.  Here's an example:

```console
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu              latest              c4ff7513909d        15 hours ago        225.4 MB
```

As you can see, the command returns the following columns:

* REPOSITORY.  The name of the repository.
* TAG.  The current tag.  We'll talk more about tags in a bit, but tags are similar to those found in git or other version control systems, an represent a specific set point in the repositories' commit history.
* IMAGE ID.  This is like the primary key for the image.  Sometimes the repository name or the tag can become detached, but you can always refer to the ID.
* CREATED.  The date the repository was created.  Note that this is different than when it was pulled.  This can help you assess how "fresh" a build is.
* VIRTUAL SIZE.  The size of the image.




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










