# Using Volumes

Docker Volumes provide a way to share storage between docker containers, or between your host OS and your containers.  Situations you might want to do this include:

* You want to keep your source code on your host environment, but launch a container where it will execute.  (This is similar to the Vagrant workflow.)
* You have a large dataset you you want to process in multiple containers.  You can use volumes to give each container access to it. 

BLAH, BLAH, BLAH.  NEED TO DO MORE INTRODUCTORY STUFF HERE.


If you're using Linux, volumes work quite.  However, if you're on a Mac or Windows, creating a volume that mounts a directory on your host machine into a container is quite tricky.  

## Introduction to volumes



## Links

* [boot2dockers official recommendation on folder sharing](boot2docker together with VirtualBox Guest Additions).  This is the offcial way to do it, but it's a bit awkward.  Basically, you create a volume, then create a container that runs a Samba server that exposes it to other containers, and then connect to it as a remote volume on your host.  (i.e., on a mac you set it up using finder)
* [boot2docker together with VirtualBox Guest Additions](https://medium.com/boot2docker-lightweight-linux-for-docker/boot2docker-together-with-virtualbox-guest-additions-da1e3ab2465c).  Describes how the author build a custom iso that included the VirtualBox additions needed to mount a local vlume into a docker container.  This isn't the official approach Docker recommends because it's too vbox-specific, but it does get towards that simple workflow people want.
* [sshfs](https://gist.github.com/codeinthehole/7ea69f8a21c67cc07293).  Yet another approach where you start a new filesystem on your host and connect to it in the container.  
