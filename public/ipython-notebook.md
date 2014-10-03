# Running Jupyter / IPython Notebooks on Docker for Mac/Windows

[Jupyter](http://jupyter.org/) is the next generation of the popular [IPython Notebooks](http://ipython.org/) suite of tools for scientific computing.  The name change to Jupyter (an amalgam of Julia, Python, and R) reflects the project's goal to become a more general purpose computational platform for any language (not just IPython). 

This chapter covers how to get Jupyter up and running in Docker.  It's a bit different in structure than the others as it's meant to be a standalone piece, and provide step-by-step instructions.  


## Install boot2docker

The first step is to install [boot2docker](https://github.com/boot2docker/boot2docker), which sets up [VirtualBox](https://www.virtualbox.org/) and a small virtual machine that can run Docker.

* [Setup for a Mac OS X](https://docs.docker.com/installation/mac/)
* [Setup for Windows](https://docs.docker.com/installation/windows/)

The chapter on [boot2docker](boot2docker.html) has more detail about this handy tool.

### Create the VM

Once boot2docker is installed, create a VM.  You only have to do this once.

```
$ boot2docker init
```

### Start the VM

```
$ boot2docker up
```

### Set the DOCKER\_HOME environment variable

Need to write instructions for both windows and mac.  Although, is this done automatically, now?

### Stopping the VM

If you're done with Docker or need to free up more memory, you can stop it like this:

```
$ boot2docker stop
```

## Add a Dockerfile

Once you've got boot2docker installed, the next step is to set up a Dockerfile for the project.  As described in the [chapter on Dockerfiles](building-images-with-dockerfiles.html), a Dockerfile is a lightweight way to describe the software on the container.  Like [many projects](https://registry.hub.docker.com/repos/stackbrew/), Jupyter has many official images you can use as the starting point for your project.  In this example, we'll use [jupyter/scipystack](https://registry.hub.docker.com/u/ipython/scipystack/).

Create a file called `Dockerfile` in the directory where your notebook files are stored.  It should have the following contents:

```
FROM ipython/scipystack

RUN mkdir -p /notebooks
ADD . /notebooks

WORKDIR /notebooks
EXPOSE 8888
CMD ipython notebook --ip=0.0.0.0 --no-browser
```

Basically, what this Dockerfiles does is add the contents of your current directory (i.e., your notebook files) into a `/notebooks` directory in the scipystack container.  It then set the current working directory to `/notebooks`, exposes port 8888 (Jupyter's default port), and starts the IPyhton notebook server itself.


## Build the image

Once you have the Dockerfile, you need to build it into an image:

```
$ docker build -t my-name/my-image-name .
```

It will probably churn away for a while the first time you run this because it needs to download all the layers in the scipystack image.  But, once it does, subsequent builds will go quickly.  

*Also, note you'll need to rebuild the image each time you change any of the files.*

## Start a container

Once you've got the image built, you're almost there.  You just need to start it running:

```
$ docker run -p 8888:8888 my-name/my-image-name
```

## Open the notebook in your browser

If all goes well, you can go to the following URL in your browser and view the notebook:

```
192.168.59.103:8888
```

Note that boot2docker only exposes a container as a local network on that IP, so you won't be able to use a typical URL like localhost. If for some reason that IP address is unavailable, boot2docker will set another one.  You can discover this by used `boot2docker ip` and then using that in your URL.  [NEED TO CLARIFY THE NETWORKING STUFF HERE]

##  Saving your work

If you're on a Mac or Windows machine, be aware that your work in the notebooks is stored only on the container, not on your local filesystem.  So, if you want to keep your work, you'll need to make sure you copy out your data.  Using git is perhaps the simplest way to do this.  First, you'll need to create a git repo on a site like GitHub or BitBucket.  Next, you can use a "shell magics" to push your changes off the repo to your new repo.

Just enter this into a cell:

```
%%bash

git init
git add .
git commit -a -m"saving all my stuff"
git remote add origin https://github.com/username/projectname
git push origin master
```

Note that if your original files (the one you added when you built the box) are also under version control in git, you will not have the same commit history, so you'll be unable to merge changes this way.  So, this is pretty much just a simple way to get you files our, but not much else.

*Docker is working on a number of ways to improve the ability to share files between containers and the host machine.  I'll be updating this section when those new features are released.  Also, this is a pretty hokey solution.  There are probably much better ones.*

## Stopping your container

When you're ready to stop your container, you can use `docker ps` and `docker kill` to stop everything.

```
$ docker ps
CONTAINER ID        IMAGE                                 COMMAND                CREATED             STATUS              PORTS                    NAMES
21761072f39f        odewahn/python-data-cookbook:latest   /bin/sh -c 'ipython    2 hours ago         Up 29 minutes       0.0.0.0:8888->8888/tcp   sick_babbage        

$ docker kill 21761072f39f
```

*NOTE: Unless you take steps to save your data as described above, you will lose your changes!*





