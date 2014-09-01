# Images: Layered filesystems

A Docker *image* represents a snapshot of a filesystem at a certain point in time.  As mentioned in the introduction, the image is composed of layers that progressively stack on top of each other; containers (running instances of an image) can share these layers among them, which is one reason Docker is so much lighter weight than a full VM, where nothing is generally shared.  

Perhaps the best way to start (after you get Docker installed, of course!) is to use "docker pull" to grab the latest release of Ubuntu (we'll talk more about where you're actually pulling from in the chapter on the Docker Hub): 

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

As you pull the image, you'll see the progress of each dependent layer being downloaded.  Once all the layers are finished downloading (and there are a LOT of layers for Ubuntu and all the versions), you can run `docker images` to get information about the images on your system.  Here's an example:

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

If you want a granular view of the layers in in an image, you can use `docker history`:

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

Each line in the history corresponds to a commit of the image's filesystem.  The values in the SIZE column add up to the corresponding VIRTUAL SIZE column for the image in `docker image`. (If you decide to double check this, remember to that the column has units, so be sure to convert all values to MB.)  

There are a couple of key things to understand about the layers in a docker images:

* They can be reused.  Docker keeps track of all the layers you've pulled.  So, if two images happen to have a layer in common (for example, if two images are built form the same base box), Docker will reuse the common parts, and only pull the diffs.
* The layers are always additive, which can lead to really big sizes if you're not careful.  For example, if you download a large file, make a commit, delete the file, and then make another commit, that large file will still be present in the layer history.  We'll come back to this idea again, so don't worry if it doesn't make too much sense right now.  Just remember that layers are always additive.

