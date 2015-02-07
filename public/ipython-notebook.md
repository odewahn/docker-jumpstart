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

## Start a container

Once you've got the image built, you're almost there.  Change int the directory that contains your notebooks and run this command.  *NB: you need Docker 1.3+ on a Mac or Linux for this to work* :

```
docker run -it           \
   -p 8888:8888          \
   -v $(pwd):/usr/data   \
   -w /usr/data          \
   ipython/scipystack    \
   ipython notebook --ip=0.0.0.0 --no-browser
```

Here's what the various options do:

* `-p 8888:8888` exposes port 8888 on the container and maps it to port 8888 on the host
* `-v $(pwd):/usr/data` maps your current working directory to the `/usr/data` directory on your container
* `-w /usr/data` sets the working directory in the container to the mapped volume where your notebooks are located
* `ipython/scipystack` is the name of the image to use
* `ipython notebook --ip=0.0.0.0 --no-browser` is the command to execute


If all goes well, you can go to the following URL in your browser and view the notebook.

* On a Mac, go to `192.168.59.103:8888`
* On Linux, go to `localhost:8888`

Note that boot2docker only exposes a container as a local network on that IP, so you won't be able to use a typical URL like localhost. If for some reason that IP address is unavailable, boot2docker will set another one.  You can discover this by used `boot2docker ip` and then using that in your URL.  [NEED TO CLARIFY THE NETWORKING STUFF HERE]


## Stopping your container

When you're ready to stop your container, you can use `docker ps` and `docker kill` to stop everything.

```
$ docker ps
CONTAINER ID        IMAGE                                 COMMAND                CREATED             STATUS              PORTS                    NAMES
21761072f39f        odewahn/python-data-cookbook:latest   /bin/sh -c 'ipython    2 hours ago         Up 29 minutes       0.0.0.0:8888->8888/tcp   sick_babbage        

$ docker kill 21761072f39f
```

*NOTE: Unless you take steps to save your data as described above, you will lose your changes!*




