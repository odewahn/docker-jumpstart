# Creating your own Docker Image

Now that we've got the basics, let's make something a tiny bit more realistic: a [Flask](http://flask.pocoo.org/) app.  In this section we'll:

* Start a basic container named "simple_flask" based on Ubuntu
* Install the dependencies
* Install the app itself (which is just the "Hello, World!" example, so it's nothing exciting)
* Commit the container to a new image for our app
* Start a new container based on our image
* Access our app using a browser

Again, the point here is not to show the best way to set up an environment, but instead to illustrate the Docker commands and what they do.  I'll code development environments in more detail later.

## Start the "simple\_flask" container

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

##  Install the dependencies

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
```

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

```console
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
```

## Install our code

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

## View the flask app on your host machine

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

## Cleaning up

If you're really excited about your "Hello World!" Flask app, you should feel free to commit it.  Otherwise, let's kill it.  First, we need to figure out its ID, which we'll do using "docker ps":

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

As you'll see, although the container has stopped running (i.e., its status has changed from Up to Exited), the container itself is still there.  In fact, unless you use the "-rm" option when you start a container, it will always leave this remnant image behind.  And, if left unchecked, after a while you'll consume your entire disk with stopped containers.  

Why is it like this, you might ask?  The answer lies in Docker's need to get a clean files state for a commit.  In a container is running, it can mean that there are open files or processes that could interfere with the ability to save the state of the filesystem.  So, rather than destroy the container automatically, Docker saves it to enable you to get a nice, clean commit image.  So, killing the image is really the same as just pausing it.  

So, to get rid of this ghost container, you need to use "docker rm":

```console
$ docker rm 2f5ada6523c4
2f5ada6523c4
```

But, what about all those other stopped containers hanging around?  Never fear! This great tip from [Jim Hoskins](http://jimhoskins.com/) in [Remove Untagged Images From Docker](http://jimhoskins.com/2013/07/27/remove-untagged-docker-images.html) shows a quick way to remove stopped containers in bulk:

``` console
$ docker rm $(docker ps -aq)
```