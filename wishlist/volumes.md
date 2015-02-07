# Using Volumes

Docker Volumes provide a way to share storage between docker containers, or between your host OS and your containers.  Situations you might want to do this include:

* You want to keep your source code on your host environment, but launch a container where it will execute.  (This is similar to the Vagrant workflow.)
* You have a large dataset you you want to process in multiple containers.  You can use volumes to give each container access to it. 

BLAH, BLAH, BLAH.  NEED TO DO MORE INTRODUCTORY STUFF HERE.


If you're using Linux, volumes work quite.  However, if you're on a Mac or Windows, creating a volume that mounts a directory on your host machine into a container is quite tricky.  

## Introduction to volumes


## Using the virtualbox guest editions to run IPython Notebooks


### Build the latest iso

This is all described here:

https://github.com/boot2docker/boot2docker/pull/534


### Create a share named "Users"

Before you can add a shared drive, stop the VM:

```
boot2docker halt
```

Next, set up a share shared folder named `Users` that points to a directory you'd like to make accessible.  For example, this command will map the `/Users` directory on the boot2docker guest VM to the `/Users/odewahn/Desktop` on your host:

```
VBoxManage sharedfolder add boot2docker-vm -name /Users -hostpath /Users/odewahn/Desktop
```

Note that you can also set up a share using the VirtualBox GUI. 

### Start the notebook container so that it can access the shared volume

Once you've set up the shared folder from your host the the VM, use the `-v` flag to set up a map the volume to the container.  The important thing to remember here is that the mapping in the container is relative to the share folder directory on your VM, not on the host.  

Here's an example.

```
docker run -it -p 8888:8888 -v /Users/jem-docker/notebook:/Users/notebook:rw -w /Users/notebook ipython/scipystack /bin/bash
```



```
docker run -it -p 8888:8888 -v /Users/python-data-cookbook/notebooks:/Users/notebook:rw -w /Users/notebook ipython/scipystack /bin/bash
```


## Volumes in 1.3

* Need to update the boot2docker client and image to 1.3

* Set the new environment variables in `.bash\_profile`

```
DOCKER_HOST=tcp://192.168.59.103:2376
DOCKER\_TLS\_VERIFY=1
DOCKER\_CERT\_PATH=/Users/odewahn/.boot2docker/certs/boot2docker-vm
```

* Set up the share and mount the volume to the container

Note that you have to stop the VM before you can create a shared drive.  Mapping the path so that it's the same on both system (i.e., at the /Users level) will keep things simpler:

```
boot2docker halt

VBoxManage sharedfolder add boot2docker-vm -name /Users -hostpath /Users

boot2docker up
```

* 

````
docker run -it -p 8888:8888 -v $(pwd):/notebooks -w /notebooks ipython/scipystack ipython notebook --ip=0.0.0.0 --no-browser 

```


## Links

* [boot2dockers official recommendation on folder sharing](boot2docker together with VirtualBox Guest Additions).  This is the offcial way to do it, but it's a bit awkward.  Basically, you create a volume, then create a container that runs a Samba server that exposes it to other containers, and then connect to it as a remote volume on your host.  (i.e., on a mac you set it up using finder)
* [boot2docker together with VirtualBox Guest Additions](https://medium.com/boot2docker-lightweight-linux-for-docker/boot2docker-together-with-virtualbox-guest-additions-da1e3ab2465c).  Describes how the author build a custom iso that included the VirtualBox additions needed to mount a local vlume into a docker container.  This isn't the official approach Docker recommends because it's too vbox-specific, but it does get towards that simple workflow people want.
* [sshfs](https://gist.github.com/codeinthehole/7ea69f8a21c67cc07293).  Yet another approach where you start a new filesystem on your host and connect to it in the container.  
* [UI for mounting shared volumes](https://github.com/boot2docker/boot2docker-cli/pull/258). This is issue that describes how this stull will all work.
* [Docker Language Stacks](https://blog.docker.com/2014/09/docker-hub-official-repos-announcing-language-stacks/).  These are official images for a variety of languages, like Python, Ruby, Go, etc.
