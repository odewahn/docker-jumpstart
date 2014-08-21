# Building images with Dockerfiles

As we saw in the Docker Walkthrough chapter, the general Docker workflow is:

* start a container based on an image in a known state
* add things to the filesystem, such as packages, codebases, libraries, files, or anything else
* commit the changes as layers to make a new image

In the walkthrough, we took a very simple approach of just starting a container interactively, running the commands we wanted (like "apt-get install" and "pip install"), and then committing the container into a new image.

In this chapter, we'll look at a more robust way to build an image.  Rather than just running commands and add files, we'll put out instructions in a special file called the *[Dockerfile](https://docs.docker.com/reference/builder/)*.  A Dockerfile is similar in concept to the recipes and manifests found in infrastructure automation (IA) tools like [Chef](http://www.getchef.com/) or [Puppet](http://puppetlabs.com/).  

Overall, a Dockerfile is much more stripped down than the IA tools, consisting of a single file with a DSL that has a handful of instructions.  The format looks like this:

```console
# Comment
INSTRUCTION arguments
```

The following table summarizes the instructions; many of these options map directly to option in the "docker run" command:


| Command    | Description 
|------------|-------------------------------------------------------
| ADD        | Copies a file from the host system onto the container
| CMD        | The command that runs when the container starts (similar to the command in *docker run*)
| ENTRYPOINT | 
| ENV        | Sets an environment variable in the new container (similar to the -e option in *docker run*)
| EXPOSE     | Opens a port (similar to the "-p" option in *docker run* )
| FROM       | The base image to use in the build.  This is mandatory and must be the first command in the file.
| MAINTAINER | An optional value for the maintainer of the script
| ONBUILD    | A command that is triggered when the image in the Dcokerfile is used as a base for another image
| RUN        | Executes a shell command and save the results on a new layer
| USER       | Sets the default user within the container (similar to the -u option  in *docker run*)
| VOLUME     | Creates a shared volume that can be shared among containers or by the host machine
| WORKDIR    | Set the default working directory for the container (similar to the -w option in *docker run) 
