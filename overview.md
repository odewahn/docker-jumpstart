# Docker Walkthrough

This chapter introduces the key ideas you'll use again and again in Docker, such as images, layers, containers, commits, tags, and so forth.  The main things to understand include:

* an *image* is a specific state of a filesystem
* an image is composed of *layers* representing changes in the filesystem at various points in time; layers are a bit like the commit history of a git repository
* a *container* is a running process that is started based on an image
* you can change the state of the filesystem on a container and commit it to create a new image
* changes in memory / state are not committed -- only changes on the filesystem

In the following section we'll walk through:

* An overview of the Docker commands
* Pulling a base image
* Running a container based on an interactive shell
* Install our app and its dependencies in the shell
* Committing the container at each major checkpoint
* Running our container in daemon mode
* Viewing our app in the browser

## Overview of the Docker commands

Like boot2docker, docker uses the git-style command format:

```console
$ docker [OPTIONS] COMMAND [arg...]
```

The following table, taken from the "docker help" command, provides a quick summary of the commands.  


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

As you scan the list (which I've left in alphabetical order), you'll notice 3 key groups of commands:

* Commands related to managing *images*, such as *images*, *build*, *save*, *rmi*, and *tag*.
* Commands related to *containers*, such as *run*, *ps*, *kill*, *restart*, *top*, and *pause*.
* Commands related to the [Docker hub](https://hub.docker.com/), such as *login*, *search*, *pull*, and *push*. (More on this in the next chapter.) 

The following sections (and other parts of this guide) show how you use the first two groups of commands to create custom images; we'll explore the final group in its own chapter.
 

## Working with Images

As a first step, let's use "docker pull" to grab the latest release of Ubuntu (we'll talk more about where you're actually pulling from in the chapter on the Docker Hub): 

```console
$ docker pull ubuntu
Pulling repository ubuntu
c4ff7513909d: Download complete 
511136ea3c5a: Download complete 
1c9383292a8f: Download complete 
9942dd43ff21: Download complete 
d92c3c92fa73: Download complete 
0ea0d582fd90: Download complete 
cc58e55aa5a5: Download complete 
...
```

As you pull the image, you'll see the progress of each dependent layer being downloaded.  Once all the layers are finished downloading (and there are a LOT of layers for Ubuntu and all the versions), you can run *docker images* to get information about the images on your system.  Here's an example:

```console
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu              14.04               c4ff7513909d        2 days ago          225.4 MB
ubuntu              latest              c4ff7513909d        2 days ago          225.4 MB
ubuntu              14.04.1             c4ff7513909d        2 days ago          225.4 MB
ubuntu              trusty              c4ff7513909d        2 days ago          225.4 MB
ubuntu              14.10               75204fdb260b        2 days ago          230.1 MB
ubuntu              utopic              75204fdb260b        2 days ago          230.1 MB
ubuntu              12.04.5             822a01ae9a15        2 days ago          108.1 MB
ubuntu              precise             822a01ae9a15        2 days ago          108.1 MB
ubuntu              12.04               822a01ae9a15        2 days ago          108.1 MB
```

As you can see, the command returns the following columns:

* REPOSITORY.  The name of the repository, which in this case is "ubuntu".
* TAG.  We'll talk more about tags in a bit, but tags are similar to those found in git or other version control systems, and represent a specific set point in the repositories' commit history.  As you can see from the list, we've pulled down a bunch of different versions of ubuntu: 14.04, 14.10, 12.04, etc.  Each of these versions is tagged with a version number, a name, and there's even a special tag called "latest" which represents the latest version.
* IMAGE ID.  This is like the primary key for the image.  Sometimes, such as when you commit a container without specifying a name or tag, the repository or the tag is *&lt;NONE&gt;*, but you can always refer to a specific image or container using its ID.
* CREATED.  The date the repository was created, as opposed to when it was pulled.  This can help you assess how "fresh" a particular build is.  Docker appears to update their master images on a fairly frequent basis.
* VIRTUAL SIZE.  The size of the image.

If you want a granular view of the layers in in an image, you can use *docker history*:

```console
$ docker history ubuntu:latest
IMAGE               CREATED             CREATED BY                                      SIZE
c4ff7513909d        7 days ago          /bin/sh -c #(nop) CMD [/bin/bash]               0 B
cc58e55aa5a5        7 days ago          /bin/sh -c apt-get update && apt-get dist-upg   32.67 MB
0ea0d582fd90        7 days ago          /bin/sh -c sed -i 's/^#\s*\(deb.*universe\)$/   1.895 kB
d92c3c92fa73        7 days ago          /bin/sh -c rm -rf /var/lib/apt/lists/*          0 B
9942dd43ff21        7 days ago          /bin/sh -c echo '#!/bin/sh' > /usr/sbin/polic   194.5 kB
1c9383292a8f        7 days ago          /bin/sh -c #(nop) ADD file:c1472c26527df28498   192.5 MB
511136ea3c5a        14 months ago                                                       0 B
```

Each line in the history corresponds to a commit of the image's filesystem.  The values in the SIZE column add up to the corresponding VIRTUAL SIZE column for the image in *docker image*. (If you decide to double check this, remember to that the column has units, so be sure to convert all values to MB.)  

There are a couple of key things to understand about the layers in a docker images:

* They can be reused.  Docker keeps track of all the layers you've pulled.  So, if two images happen to have a layer in common (for example, if two images are built form the same base box), Docker will reuse the common parts, and only pull the diffs.
* The layers are always additive, which can lead to really big sizes if you're not careful.  For example, if you download a large file, make a commit, delete the file, and then make another commit, that large file will still be present in the layer history.  We'll come back to this idea again, so don't worry if it doesn't make too much sense right now.  Just remember that layers are always additive.

## Working with Containers

So, now that you've got an image, we use *docker run* to start a new container.  Here's the format for the command:

```console
$ docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
````

The command consists of:

* OPTIONS.  There are a LOT of options for the run command.  We'll cover this in a bit more depth in the next shortly. 
* IMAGE.  The name of the image that the container should start from.
* COMMAND.  The command that should run *on the container when it starts*.  

So, let's try a simple command to to start a new container using the latest version of Ubuntu.  Once this container starts, you'll be at a bash shell where you can do "cat /etc/os-release" to see the release information about the OS:

```console
$ docker run -it ubuntu:latest /bin/bash
root@4afa46473802:/# cat /etc/os-release 
NAME="Ubuntu"
VERSION="14.04.1 LTS, Trusty Tahr"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 14.04.1 LTS"
VERSION_ID="14.04"
HOME_URL="http://www.ubuntu.com/"
SUPPORT_URL="http://help.ubuntu.com/"
BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
```

Next, type this command in your running container to make a simple change on the filesystem:

```console
root@4afa46473802:/# echo "hello world" > /tmp/hello.txt
```

Now that you've verified the OS release and created a test file, here's a bit more detail about the options we've used in the RUN command:

* The OPTIONS "-it" tell Docker to make the container interactive and provide a tty (i.e., attach a terminal)
* The IMAGE is the "ubuntu:latest" image we pulled down.  Note the *repository:tag* format, which is used throughout docker.  By default, the "latest" tag is used, but if we were to use "ubuntu:precise" we'd be using Ubuntu 12.04.
* The COMMAND is "/bin/bash", which starts a shell where you can log in

The cool thing is that once you're in an interactive docker shell, you can do anything you want, such as installing packages (via apt-get), code bases (using git), or dependencies (using pip, gem, npm, or any other package manager du jour.)  You can then commit the container at any point to create a new image that has your changes.  

To see this in action, open a new terminal on your host machine and use "docker ps" command to see some information about the running container:

```console
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
4afa46473802        ubuntu:14.04        /bin/bash           17 hours ago        Up 39 seconds                           jolly_perlman
```

This command will tell you:

* The CONTAINER\_ID, a unique identifier you can use to refer to the container in other commands (this is kind of like a process id in Linux)
* The IMAGE that was used when the container started
* The COMMAND used to start the container
* The time the underlying image was CREATED
* The uptime STATUS
* The exposed ports (lot's more on this!)
* A human readable NAME that you can use in place of the ID (this is something you can also assign yourself, which we'll get into shortly.)

Now try the "docker diff" command:

```console
$ docker diff jolly_perlman
C /tmp
A /tmp/hello.txt
```

As you can see, this command returns the changes that have happened on the filesystem, which in our case is that we've changed the "/tmp" directory by adding a "hello.txt" file.  Now let's commit the changes we've made into a new container:

```console
$ docker commit -m "Adding hello world" jolly\_perlman hello\_docker
5af244124dd8656f553731b9c60e21872f34e7c5532b85dcc7052fcf5e2cf2ef
```

Taking another look at *docker images* shows the effect; we now have a new image called "hello\_docker" that includes our "/tmp/hello.txt" file.  Taking a quick look at the container's history shows how our change added a whopping new 12 byte layer on top of the original layers of ubuntu:latest.

```console
$ docker history hello_docker
IMAGE               CREATED             CREATED BY                                      SIZE
5af244124dd8        18 hours ago        /bin/bash                                       12 B
c4ff7513909d        2 days ago          /bin/sh -c #(nop) CMD [/bin/bash]               0 B
cc58e55aa5a5        2 days ago          /bin/sh -c apt-get update && apt-get dist-upg   32.67 MB
0ea0d582fd90        2 days ago          /bin/sh -c sed -i 's/^#\s*\(deb.*universe\)$/   1.895 kB
d92c3c92fa73        2 days ago          /bin/sh -c rm -rf /var/lib/apt/lists/*          0 B
9942dd43ff21        2 days ago          /bin/sh -c echo '#!/bin/sh' > /usr/sbin/polic   194.5 kB
1c9383292a8f        2 days ago          /bin/sh -c #(nop) ADD file:c1472c26527df28498   192.5 MB
511136ea3c5a        14 months ago                                                       0 B
```

Finally, we can kill our docker process like this:

```console
$ docker kill jolly_perlman
jolly_perlman
```

If you do *docker ps* again, you'll see that the process has stopped. 


## Creating a simple docker app

Now that we've got the basics, let's make something a tiny bit more realistic: a [Flask](http://flask.pocoo.org/) app.  In this section we'll:

* Start a basic container named "simple_flask" based on Ubuntu
* Install the dependencies
* Install the app itself (which is just the "Hello, World!" example, so it's nothing exciting)
* Commit the container to a new image for our app
* Start a new container based on our image
* Access our app using a browser

Again, the point here is not to show the best way to set up an environment, but instead to illustrate the Docker commands and what they do.  I'll code development environments in more detail later.

### Start the "simple\_flask" container

Our first step is to start a new container based on "ubuntu:latest."  As we did in the previous example, we'll make it interactive using the "-it" options, but this time we'll give it our own name:

```console
$ docker run -it --name="simple_flask" ubuntu:latest /bin/bash
``` 

The nice thing about using a name is that you can use it in other docker commands in place of the container id.  For example, you can use this command to see the top process running on the container. (Also, it's worth noting that you need to run this on a terminal on your host, not in the container itself.)  

```console
$ docker top simple_flask
PID                 USER                COMMAND
969                 root                /bin/bash
```

###  Install the dependencies

Now that we've got the container running, we install the dependencies we need.  First, since Flask is a Python micro framework, let's check out what version of Python we have:

```console
root@2f5ada6523c4:/home# python --version
bash: python: command not found
```

"Wait a minute!" you might be asking yourself, "What gives? Shouldn't Python be installed on Ubuntu?"  And, the answer is "Yes," on a "real" Ubuntu distribution it is.  But, to save on size, the Docker base images are stripped down versions of the official images.  This is important to keep this in mind as you build new containers: don't take it for granted that everything on the official distribution will be present on the Docker base image.

So, let's get a minimal python environment going.  First, we'll update our apt repository:

```console
root@2f5ada6523c4:/home# apt-get update
```

This will churn away for a while updating various packages.  Once it's complete, install python and pip:

```console
root@2f5ada6523c4:/home# apt-get install -y python python-pip wget
```

This churns even longer, but once it's done, you can install Flask:

```console
root@2f5ada6523c4:/home# pip install Flask
```

Whew.  That's a lot of stuff to install, so let's commit the image with a simple commit message:

```console
$ docker commit -m "installed python and flask" simple_flask
c21c062b086aeebc6dd994710eef420ad087ce7bc4cbe03109089bd425497035

So, now if you check your "docker images" you'll see the new repository:

```console
admins-air-5:~ odewahn$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
<none>              <none>              c21c062b086a        2 days ago          385.7 MB
ubuntu              latest              c4ff7513909d        7 days ago          225.4 MB
```

But, ugh, it doesn't have a name -- it just shows up as "&lt;none&gt;".  We can use *docker tag* to give our image a nice name, like this:

```console
$ docker tag c21c062b086a simple_flask
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
simple_flask        latest              c21c062b086a        2 days ago          385.7 MB
ubuntu              latest              c4ff7513909d        7 days ago          225.4 MB
```

Finally, taking a quick look at the history shows that we've loaded about 160MB worth of new stuff onto our image:

$ docker history simple_flask
IMAGE               CREATED             CREATED BY                                      SIZE
c21c062b086a        2 days ago          /bin/bash                                       160.3 MB
c4ff7513909d        7 days ago          /bin/sh -c #(nop) CMD [/bin/bash]               0 B
cc58e55aa5a5        7 days ago          /bin/sh -c apt-get update && apt-get dist-upg   32.67 MB
0ea0d582fd90        7 days ago          /bin/sh -c sed -i 's/^#\s*\(deb.*universe\)$/   1.895 kB
d92c3c92fa73        7 days ago          /bin/sh -c rm -rf /var/lib/apt/lists/*          0 B
9942dd43ff21        7 days ago          /bin/sh -c echo '#!/bin/sh' > /usr/sbin/polic   194.5 kB
1c9383292a8f        7 days ago          /bin/sh -c #(nop) ADD file:c1472c26527df28498   192.5 MB
511136ea3c5a        14 months ago                                                       0 B

### Install our code

Let's get our code installed.  First, fire up a new container:


```console
$ docker run -it -p 5000:5000 simple_flask /bin/bash
```

As before, we'll use the "-it" options to make it interactive, but we're now adding a couple of new settings:

* The "-p 5000:5000" option exposes port 5000 on the container and routes it to port 5000 on the host.  If you're running Docker on a Mac, the confusing thing to remember is that from the Docker container's perspective, the virtual machine is the host, so you'll also need to expose port 5000 from the VM in order to see it on your "real" host OS.  We'll cover this [Inception](http://en.wikipedia.org/wiki/Inception)-like step in the next section.
* The "-w /home" option sets the working directory used when the container starts.  Note that this must be an absolute path.

Once we're in, we'll grab our simple hello world file from this projects github repo:

```console
# wget https://raw.githubusercontent.com/odewahn/docker-jumpstart/master/examples/hello.py
--2014-08-17 05:38:02--  https://raw.githubusercontent.com/odewahn/docker-jumpstart/master/examples/hello.py
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 199.27.78.133
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|199.27.78.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 149 [text/plain]
Saving to: 'hello.py'

100%[===========================================================================>] 149         --.-K/s   in 0s      

2014-08-17 05:38:02 (1.34 MB/s) - 'hello.py' saved [149/149]

root@2f5ada6523c4:/home# cat hello.py
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello World!'

if __name__ == '__main__':
    app.run(host='0.0.0.0')
```

Finally, once you've got the file, start the app.  You should see something like this:

```console
root@2f5ada6523c4:/home# python hello.py 
 * Running on http://127.0.0.1:5000/
```

### View the flask app on your host machine

We're ready for the big reveal.  On a terminal on your host, try this command:

```console
$ curl localhost:5000
```

If you're running this on Linux as your host OS, you should see "Hello World!" printed.  If you're running it on boot2docker on a Mac or Windows, you most likely got this error:

```console
curl: (7) Failed connect to localhost:5000; Connection refused
```

As I mentioned earlier, when you started the container and told it to expose port 5000, it exposed it *on the virtual machine*, not on your host.  As described in the chapter on boot2docker, you need to use VBoxManage to open the port from the VM onto your host, like this:

```console
$ VBoxManage controlvm boot2docker-vm natpf1 "flask-server,tcp,127.0.0.1,5000,,5000"
```

You should now be able to connect to the simple app, like this:

```console
$ curl localhost:5000
Hello World!
```

### Cleaning up

If you're really excited about your "Hello World!" Flask app, you should feel free to commit it.  Otherwise, let's kill is.  First, we need to figure out its ID, which we'll do using "docker ps":

```console
$ docker ps -a
CONTAINER ID        IMAGE                 COMMAND             CREATED             STATUS                      PORTS                    NAMES
2f5ada6523c4        simple_flask:latest   /bin/bash           3 days ago          Up 34 minutes               0.0.0.0:5000->5000/tcp   nostalgic_goodall                           simple_flask
```

We can kill our simple_flask app running as container "2f5ada6523c4" like this:

```console
$ docker kill 2f5ada6523c4
2f5ada6523c4
```

To verify it's gone, lets run "docker ps" again, but this time add the "-a" option so that we see all containers: 

```console
$ docker ps -a
CONTAINER ID        IMAGE                 COMMAND             CREATED             STATUS                      PORTS               NAMES
2f5ada6523c4        simple_flask:latest   /bin/bash           3 days ago          Exited (-1) 2 seconds ago                       nostalgic_goodall   
55ba5b67cb28        simple_flask:latest   /bin/bash           3 days ago          Exited (0) 40 minutes ago                       dreamy_mayer        
6029929a1e53        ubuntu:latest         /bin/bash           3 days ago          Exited (0) 13 hours ago                         naughty_leakey      
ef9bd2df07c6        ubuntu:latest         /bin/bash           4 days ago          Exited (0) 13 hours ago                         simple_flask        
```

As you'll see, although the container has stopped running (i.e., it's status has changed from Up to Exited), the container itself is still there.  In fact, unless you use the "-rm" option when you start a container, it will always leave this remnant image behind.  And, if left unchecked, after a while you'll consume your entire disk with stopped containers.  

Why is it like this, you might ask?  The answer lies in Docker's need to get a clean files tate for a commit.  In a container is running, it can mean that there are open files or processes that could interfere with the ability to save the state of the filesystem.  So, rather than destroy the container automatically, Docker saves its state so that you can get a nice, clean commit image.  So, killing the image is really the same as just pausing it.  

So, to get rid of this ghost container, you need to use "docker rm":

```console
$ docker rm 2f5ada6523c4
2f5ada6523c4
```

But, what about all those other stopped containers hanging around?  Never fear! This great tip from [Jim Hoskins](http://jimhoskins.com/) in [Remove Untagged Images From Docker](http://jimhoskins.com/2013/07/27/remove-untagged-docker-images.html) shows a quick way to remove stopped containers in bulk:

``` console
$ docker rm $(docker ps -aq)
```

## Docker RUN Quick Reference

Hopefully, this chapter has given you a good idea of the key workflows around containers and images.  Although the example is contrived, the basic workflow is something you'll use again and again.  In the next chapter on Dockerfiles I'll present a way to automate this process even further using a Dockerfile.  

Until then, I'll conclude with this quick reference guide for the Docker RUN command.  Taken directly from *docker help*, this table shows how RUN is one of the most feature-rich commands in Docker. 



| Option                   |  Description
|--------------------------|-----------------------------------------------------------------------|
| -a, --attach=[]          |  Attach to stdin, stdout or stderr.
| -c, --cpu-shares=0       |  CPU shares (relative weight)
| --cidfile=""             |  Write the container ID to the file
| --cpuset=""              |  CPUs in which to allow execution (0-3, 0,1)
| -d, --detach=false       |  Detached mode: Run container in the background, print new container id
| --dns=[]                 |  Set custom dns servers
| --dns-search=[]          |  Set custom dns search domains
| -e, --env=[]             |  Set environment variables
| --entrypoint=""          |  Overwrite the default entrypoint of the image
| --env-file=[]            |  Read in a line delimited file of ENV variables
| --expose=[]              |  Expose a port from the container without publishing it to your host
| -h, --hostname=""        |  Container host name
| -i, --interactive=false  |  Keep stdin open even if not attached
| --link=[]                |  Add link to another container (name:alias)
| --lxc-conf=[]            |  (lxc exec-driver only) Add custom lxc options --lxc-conf="lxc.cgroup.cpuset.cpus = 0,1"
| -m, --memory=""          |  Memory limit (format: <number><optional unit>, where unit = b, k, m or g)
| --name=""                |  Assign a name to the container
| --net="bridge"           |  Set the Network mode for the container
|                          |    'bridge': creates a new network stack for the container on the docker bridge
|                          |    'none': no networking for this container
|                          |    'container:<name|id>': reuses another container network stack
|                          |    'host': use the host network stack inside the container.  Note: the host mode gives the container full access to local system services such as D-bus and is therefore considered insecure.
| -P, --publish-all=false  |  Publish all exposed ports to the host interfaces
| -p, --publish=[]         |  Publish a container's port to the host
|                          |    format: ip:hostPort:containerPort | ip::containerPort | hostPort:containerPort
|                          |    (use 'docker port' to see the actual mapping)
| --privileged=false       |  Give extended privileges to this container
| --rm=false               |  Automatically remove the container when it exits (incompatible with -d)
| --sig-proxy=true         |  Proxify received signals to the process (even in non-tty mode). SIGCHLD is not proxied.
| -t, --tty=false          |  Allocate a pseudo-tty
| -u, --user=""            |  Username or UID
| -v, --volume=[]          |  Bind mount a volume (e.g., from the host: -v /host:/container, from docker: -v /container)
| --volumes-from=[]        |  Mount volumes from the specified container(s)
| -w, --workdir=""         |  Working directory inside the container




