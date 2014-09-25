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

You can use the GUI, but you can also use this command:

```
VBoxManage sharedfolder add boot2docker-vm -name /Users2 -hostpath /Users/odewahn/Desktop
```


### Start the notebook container so that it can access the shared volume

docker run -p 8888:8888 -v /Users/notebook-test:/Users/notebook-test:rw -w /Users/notebook-test odewahn/jem-tutorial ipython notebook --ip=0.0.0.0 --port=8888 --pylab=inline --no-browser

## Links

* [boot2dockers official recommendation on folder sharing](boot2docker together with VirtualBox Guest Additions).  This is the offcial way to do it, but it's a bit awkward.  Basically, you create a volume, then create a container that runs a Samba server that exposes it to other containers, and then connect to it as a remote volume on your host.  (i.e., on a mac you set it up using finder)
* [boot2docker together with VirtualBox Guest Additions](https://medium.com/boot2docker-lightweight-linux-for-docker/boot2docker-together-with-virtualbox-guest-additions-da1e3ab2465c).  Describes how the author build a custom iso that included the VirtualBox additions needed to mount a local vlume into a docker container.  This isn't the official approach Docker recommends because it's too vbox-specific, but it does get towards that simple workflow people want.
* [sshfs](https://gist.github.com/codeinthehole/7ea69f8a21c67cc07293).  Yet another approach where you start a new filesystem on your host and connect to it in the container.  
* [UI for mounting shared volumes](https://github.com/boot2docker/boot2docker-cli/pull/258). This is issue that describes how this stull will all work.
* [Docker Language Stacks](https://blog.docker.com/2014/09/docker-hub-official-repos-announcing-language-stacks/).  These are official images for a variety of languages, like Python, Ruby, Go, etc.
