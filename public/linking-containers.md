# Creating multi-container apps

Even the simplest apps have many parts and services: a database, a job processing queue, an app server, and a caching layer.  

Docker allows you to run each of these services in it's own container and then link them together so that they can communicate.

## docker-compose

[docker-compose](https://docs.docker.com/compose/) is a tool for managing multiple container apps in Docker. It's kind of like foreman 
