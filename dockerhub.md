# Sharing images on DockerHub

## Create an account

## Searching for images


Let's get started with the example.  First, we'll need a base image, so let's do a "docker search ubuntu"

```console
$ docker search ubuntu
NAME                                             DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
ubuntu                                           Official Ubuntu base image                      525       [OK]       
stackbrew/ubuntu                                 Official Ubuntu base image                      40        [OK]       
dockerfile/ubuntu                                Trusted Ubuntu (http://www.ubuntu.com/) Bu...   19                   [OK]
crashsystems/gitlab-docker                       A trusted, regularly updated build of GitL...   19                   [OK]
ubuntu-upstart                                   Upstart is an event-based replacement for ...   10        [OK]       
dockerfile/ubuntu-desktop                        Trusted Ubuntu Desktop (LXDE) (http://lxde...   9                    [OK]
lukasz/docker-scala                              Dockerfile for installing Scala 2.10.3, Ja...   7                    [OK]
litaio/ruby                                      Ubuntu 14.04 with Ruby 2.1.2 compiled from...   7                    [OK]
mbentley/ubuntu-django-uwsgi-nginx                                                               6                    [OK]
cmfatih/phantomjs                                PhantomJS [ phantomjs 1.9.7, casperjs 1.1....   5                    [OK]
zmarcantel/cassandra                             Cluster/Standalone DataStax Cassandra on U...   4                    [OK]
suttang/gollum                                   The gollum wiki installed on ubuntu             4                    [OK]
...
```

As of this writing, we get over 1300 results.  

### Image Namespaces

* root level
* user level
* "private" repo?

## Pulling an image

## Pushing an image

## A tour of interesting images