# Building images with Dockerfiles

As we saw in the Docker Walkthrough chapter, the general Docker workflow is:

* start a container based on an image in a known state
* add things to the filesystem, such as packages, codebases, libraries, files, or anything else
* commit the changes as layers to make a new image

In the walkthrough, we took a very simple approach of just starting a container interactively, running the commands we wanted (like "apt-get install" and "pip install"), and then committing the container into a new image.

In this chapter, we'll look at a more robust way to build an image.  Rather than just running commands and add files, we'll put out instructions in a special file called the *[Dockerfile](https://docs.docker.com/reference/builder/)*.  A Dockerfile is similar in concept to the recipes and manifests found in infrastructure automation (IA) tools like [Chef](http://www.getchef.com/) or [Puppet](http://puppetlabs.com/).  

In general, though, a Dockerfile is much more stripped down than the  IA tools. It's bascially a single file with a simple DSL with a handfule of commands that have the following format:

```console
# Comment
INSTRUCTION arguments
```

 





These are summarized in the following table:


| Command    | Description 
|------------|-------------------------------------------------------
| FROM       | The base image to use in the build.  This is a mandatory and must be the first command in the file.
| MAINTAINER |
| RUN        | 
| CMD        |
| EXPOSE     |
| ENV        | 
| ADD        |
| ENTRYPOINT |
| VOLUME     |
| USER       |
| WORKDIR    |
| ONBUILD    | 