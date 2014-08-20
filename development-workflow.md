# Setting up a Development Workflow

This chapter will describe how to use Docker in a development workflow that is similar to the one done in Vagrant.

This seems to only work on Linux -- maybe it's not worth talking about quite yet?


```
docker run -i -t -v $(pwd):/opt/andrew ubuntu /bin/bash
```





VBoxManage controlvm boot2docker-vm natpf1 "ipython-notebook,tcp,127.0.0.1,8888,,8888"
docker run -it -p 8888:9000 -e "PASSWORD=MakeAPassword" -v $(pwd):/opt/local -w /opt/local ipython/notebook


docker run -d -p localhost:8888:8888 -e "PASSWORD=MakeAPassword" ipython/notebook



This will remove all containers:

```console
docker rm $(docker ps -aq)
```