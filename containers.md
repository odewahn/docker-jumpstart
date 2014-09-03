# Containers: Running instances of an Image

A container is a running image that you start with the `docker run` command, like this:

```console
$ docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

The command consists of:

* OPTIONS.  There are a LOT of options for the run command.  We'll cover this in a bit more depth in the next shortly. 
* IMAGE.  The name of the image that the container should start from.
* COMMAND.  The command that should run *on the container when it starts*.  

So, let's try a simple command to to start a new container using the latest version of Ubuntu.  Once this container starts, you'll be at a bash shell where you can do `cat /etc/os-release` to see the release information about the OS:

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

Now try the `docker diff` command:

```console
$ docker diff jolly_perlman
C /tmp
A /tmp/hello.txt
```

As you can see, this command returns the changes that have happened on the filesystem, which in our case is that we've changed the "/tmp" directory by adding a "hello.txt" file.  Now let's commit the changes we've made into a new container:

```console
$ docker commit -m "Adding hello world" jolly\_perlman hello\_docker
5af244124dd8656...
```

Taking another look at `docker images` shows the effect; we now have a new image called "hello\_docker" that includes our "/tmp/hello.txt" file.  Taking a quick look at the container's history shows how our change added a whopping new 12 byte layer on top of the original layers of ubuntu:latest.

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

## Docker RUN Quick Reference

The next chapter will provide much more detail on how to create your own containers and images using *docker run*, one of the most feature-rich commands in Docker.  Taken directly from *docker help*, this table is a useful quick reference you can refer to as you go further with Docker. 


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
|                          |    'container:<name or id>': reuses another container network stack
|                          |    'host': use the host network stack inside the container.  Note: the host mode gives the container full access to local system services such as D-bus and is therefore considered insecure.
| -P, --publish-all=false  |  Publish all exposed ports to the host interfaces
| -p, --publish=[]         |  Publish a container's port to the host
|                          |    format: ip:hostPort:containerPort or ip::containerPort or hostPort:containerPort
|                          |    (use 'docker port' to see the actual mapping)
| --privileged=false       |  Give extended privileges to this container
| --rm=false               |  Automatically remove the container when it exits (incompatible with -d)
| --sig-proxy=true         |  Proxify received signals to the process (even in non-tty mode). SIGCHLD is not proxied.
| -t, --tty=false          |  Allocate a pseudo-tty
| -u, --user=""            |  Username or UID
| -v, --volume=[]          |  Bind mount a volume (e.g., from the host: -v /host:/container, from docker: -v /container)
| --volumes-from=[]        |  Mount volumes from the specified container(s)
| -w, --workdir=""         |  Working directory inside the container




