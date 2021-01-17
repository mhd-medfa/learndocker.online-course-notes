# learndocker.online course notes
**My personal notes of the amazing training program learndocker.online by Julian Fahrer.**

---------------------------------------------------

Container is the runtime construct of a container image & the container image packages the application and all the application dependencies.
First of all you need to know that docker containers are processes in the system and these containers run in isolation

====================================================

To run a docker container and open it in a shell:

`$ docker container run -it alpine sh`

-or-

`$ docker run -it alpine sh`

ALSO you can run the container in the detach mode by:

`$ docker run -itd alpine sh`

LATER if you want to attach to it, you can use:

`$ docker container attach NAME`

run this command:

`/ # hostname`

If you want to exit from shell session you can click:

`ctrl+D` in case that you want to terminate.

 -or-

`ctrl+P & ctrl+Q` if you want to keep it up running in the background

if you want to make sure if contanier is still running:

`$ docker container ls`

You'll find `UP xx seconds` which means container is still up and running

-or- 

if you want to get all containers whether it's running or not

`$ docker container ls -a`



If you want to attach again to the running container:

`$ docker attach NAME`

-or-

`$ docker container attach NAME`

Now if you run the command:

`/ # hostname`

-> you'll find that you are again in the same container with the same hostname


Now if you want to run an `Exited` container you can run the following commands:

`$ docker container start NAME`

`$ docker container attach NAME`

now if you want to stop it you can either press:

`ctrl+D`

-or-

`ctrl+P & ctrl+Q` , then:

`$ docker container stop NAME`


===========================================================

In order to remove a container, first you can get the container id by running:

`$ docker container ls -a`

or if you want to list containers ids without other information:

`$ docker container ls -aq`

Now you run:

`$ docker container rm CONTAINER_ID`

However if you want to delete all containers:

`$ docker container rm $(docker container ls -aq)`


================================================================

Hints:

If you want to create a container that will be removed and deleted automatically once you exit pass `--rm` flag when you run the container like this:

`$ docker container run --rm hello-world`

Note: the previous command won't remove the image itself but the created container.


Now if you want to create a container and give it a name (instead of the random name):

`$ docker container run --name my_container1 hello-world`

=================================================================

Publishing a service:

if you want to publish a service like running an nginx webserver you need to use -p tag to map ports like this:

`$ docker container run -p 80:80 nginx`

Now if you swith to the browser and go to 'localhost' you should have the welcome page of Nginx.

That means we successfully talked to a service that is Exposed and provided by a container.

If you want to change the system port (for example to `8080`) that listens to docker service:

`$ docker container run -p 8080:80 nginx`

Now in order to access to it you need to open 'localhost:8080' in the browser and it should successfully run.

==================================================================
Since we know how to access a webserver that runs in a docker container let's try to make the nginx service serve a web content like a simple static content (html page).
That will be using `--volume` flag. In our example we will run the example in the documentation of nginx on docker hub.

**index.html**

```<!DOCTYPE html>
<html>
  <body>
    <h1>This is a simple test</h1>
    <p>This is a simple test</p>
  </body>
</html>
```


`$ docker container run -p 80:80 -v index.html:/usr/share/nginx/html nginx`

However we should know that a simple Dockerfile can be used to generate a new image that includes the necessary content(which is a much cleaner solution thant the bind mount above).

**Dockerfile**
```FROM nginx
COPY static-html-directory /usr/share/nginx.html
```

run:

`$ docker build -t some-content-nginx .`

`$ docker run --name some-nginx -d some-content-nginx`

=======================================================

To show how containers run in isolation let's run a container and show processes status using ps:

`$ docker container run -it alpine sh`

`/ # ps -ef`

output is gonna be something like below:

```PID   USER     TIME  COMMAND
    1 root      0:00 sh
    7 root      0:00 ps -ef
/ # 
```

You notice that all you get is the processes that's running in the context of the container not the processes of the host operating system

Now if you run another container you might get the same processes but if you run a new process it won't be existed in the other

There are other forms of isolations as well, for example, the networking. If you run:

Container_1:

```/ # ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
6: eth0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
/ # 
```


Container_2:
```/ # ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
8: eth0@if9: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
/ # 
```

You can notice both of the containers have the same network interface (eth0), but they have different ip addresses (172.17.0.2 and 172.17.0.3)

ALSO containers isolated on the level of File System:

Container_1:
```/ # df -h
Filesystem                Size      Used Available Use% Mounted on
overlay                  71.6G     63.1G      4.8G  93% /
tmpfs                    64.0M         0     64.0M   0% /dev
tmpfs                     7.7G         0      7.7G   0% /sys/fs/cgroup
shm                      64.0M         0     64.0M   0% /dev/shm
/dev/nvme0n1p6           71.6G     63.1G      4.8G  93% /etc/resolv.conf
/dev/nvme0n1p6           71.6G     63.1G      4.8G  93% /etc/hostname
/dev/nvme0n1p6           71.6G     63.1G      4.8G  93% /etc/hosts
tmpfs                     7.7G         0      7.7G   0% /proc/asound
tmpfs                     7.7G         0      7.7G   0% /proc/acpi
tmpfs                    64.0M         0     64.0M   0% /proc/kcore
tmpfs                    64.0M         0     64.0M   0% /proc/keys
tmpfs                    64.0M         0     64.0M   0% /proc/timer_list
tmpfs                    64.0M         0     64.0M   0% /proc/sched_debug
tmpfs                     7.7G         0      7.7G   0% /proc/scsi
tmpfs                     7.7G         0      7.7G   0% /sys/firmware
/ # 
```

Container_2:
```/ # df -h
Filesystem                Size      Used Available Use% Mounted on
overlay                  71.6G     63.1G      4.8G  93% /
tmpfs                    64.0M         0     64.0M   0% /dev
tmpfs                     7.7G         0      7.7G   0% /sys/fs/cgroup
shm                      64.0M         0     64.0M   0% /dev/shm
/dev/nvme0n1p6           71.6G     63.1G      4.8G  93% /etc/resolv.conf
/dev/nvme0n1p6           71.6G     63.1G      4.8G  93% /etc/hostname
/dev/nvme0n1p6           71.6G     63.1G      4.8G  93% /etc/hosts
tmpfs                     7.7G         0      7.7G   0% /proc/asound
tmpfs                     7.7G         0      7.7G   0% /proc/acpi
tmpfs                    64.0M         0     64.0M   0% /proc/kcore
tmpfs                    64.0M         0     64.0M   0% /proc/keys
tmpfs                    64.0M         0     64.0M   0% /proc/timer_list
tmpfs                    64.0M         0     64.0M   0% /proc/sched_debug
tmpfs                     7.7G         0      7.7G   0% /proc/scsi
tmpfs                     7.7G         0      7.7G   0% /sys/firmware
/ # 
```
you notice they various mount points but if you created a file in one container:

Container_1:

```/ # touch /foo
/ # ls -l /foo
-rw-r--r--    1 root     root             0 Jan  8 16:07 /foo
/ # 
```
it won't be existed in the other container:
Container_2:

```/ # ls -l /foo
ls: /foo: No such file or directory
/ # 
```

**IMPORTANT:** Docker achieves this isolation by using a featurn of the linux kernel called `namespaces`.

So far we have dealt with 3 namespaces: `net namespace` (for the networking), `PID namespace` (for the processes), and the `mount(mnt) namespace` (for the file system).
  
==============================================================================

**Container Communication**

Since each container has its own ip address and they are in the same network, we can reach one from the other using its ip address (ping ip_address). However, finding ip address of the containers can be tedious and it would be nice if we can refer to container by its name.
In fact we can do that by using --link tag, suppose we have two containers with names c1 & c2:

`$ docker container run -it c2 --link c1 alpine sh`

`/ # ping c1`

HOWEVER be aware that links --link are deprecated and we have shown them here just to know they exist and still being used in many examples on the internet.

The other modern alternative is to use User defined networks.

```$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
bfdd2558e918   bridge    bridge    local
71f1000118db   host      host      local
9392907c3085   none      null      local
```

If you created a container without specifying the nework, then 'bridge' will be the default network that'll be attached to.

We can now add a user defined network to this list with:

```$ docker network create test
8b6790ee80de3285780ab6017ee93c15be587fd36417a2ed3681cb4043d27237
$docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
bfdd2558e918   bridge    bridge    local
71f1000118db   host      host      local
9392907c3085   none      null      local
8b6790ee80de   test      bridge    local
```

Now we need to create containers and attach them to the network

`$docker container run --rm -it --name c1 --network test alpine sh`

`$docker container run --rm -it --name c2 --network test alpine sh`

Now we can easily make one container talk to the other, simply by calling its name:
```/ # ping c1
PING c1 (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.212 ms
64 bytes from 172.18.0.2: seq=1 ttl=64 time=0.139 ms
64 bytes from 172.18.0.2: seq=2 ttl=64 time=0.141 ms
64 bytes from 172.18.0.2: seq=3 ttl=64 time=0.135 ms
...
```

To clean up:

`$ docker network rm test`

===================================================================

Docker has docker client and docker daemon

we talk to docker daemon using docker client CLI

In docker client we have two kinds of commands:

**Docker Management Commands**

-and-

**Docker Commands**

Most of the Docker Commands are just aliases (there are some exceptions like 'docker version' command) to Docker Management Commands.

In order to read details and get help about any command or its flags all you need to do is to write the command then tail it with the flag `--help`:
```$ docker --help
$ docker container ls --help
$ docker network create --help
...
```

In general you should favor the Docker Management commands over the Docker Commands.

for example it's better to use:

`$ docker container ls`

instead of 

`$ docker ls`

===========================================================

Docker Images

list all images in our system:

`$ docker image ls`

All images have Repository names, TAGs, tag refers to the version of the image. In case you don't specify it explicitly, docker will automatically use the tag 'latest'.
Additionally, each image has Image id, size, and the created column that tells us when the image has been created.

to remove the image:

`$ docker image rm alpine`

you can optinally refers the tag (if you don't, docker will assume you mean tag latest):

`$ docker image rm alpine:latest`

You already know that you cannot delete an image if it's still in use by at least one container.

to download an image:

`$ docker image pull hello-world:latest`


an image repository can contain multiple images, each image has different tag and a unique `IMAGE ID`. Although the image has a unique `IMAGE ID`, it could have multiple `TAGS`.

When you look for an image on Docker Hub, what you dealing with actually is Image Registry and Docker Hub is the frontend of the Image Registry.
Image Registry has multiple repositories, and each repository could have multiple images.
Docker Hub is not the only Image Registry out there but it's the default.


=====================================================

**Creating Images**


Let's suppose that we need to have an alpine modified image that supports bash (the default alpine image don't support bash).

To do that we write a Dockerfile.

Dockerfiles describe how to build an image step by step.

With the `FROM` instruction you can base your image on an already existing image.

With the `RUN Dockerfile` instruction you can execute a shell command in the context of the image while building it.

Note: apk is the package manager of Alpine.

**Dockerfile**
```FROM alpine:latest
RUN apk update
RUN apk add bash

CMD bash
```

`$ docker image build -t myalpine:latest .`

`$ docker container run -it myalpine:latest`

To write a comment in Dockerfile, it has to start from the beginning of the line.

`# This is a valid comment`

`FROM nginx:latest # This is not a comment (docker build will fail)`

==========================================================

Pushing images:

To push an image first you need to specify its namespace like this(instead of docker_course you need to use a permitted namespace you have "your `Docker ID` is a permitted namespace"):

`$ docker image tag myalpine:latest docker_course/myalpine:latest`

Where `$ docker image tag <CURRENT_REPOSITORY>:<CURRENT_TAG> <NEW_REPOSITORY>:<NEW_TAG>` allows you to apply and additional tag to an image.

Now you can push your image:

```$ docker image push docker_course/myalpine:latest

The push refers to repository [docker.io/docker_course/myalpine]
3678be0a0f9d: Pushed 
b2ce6eb35008: Pushed 
777b2c648970: Pushed 
latest: digest: sha256:c3d2b67bfe7c52a9841d911cf7fc4948939bb84df71775ff32c5ff0365cae80c size: 949
```

=============================================================

Example:

```## Base image (FROM)
debian:buster-slim

## Run the following commands (RUN)
apt-get update
apt-get install -y nginx

## Startup command (CMD)
nginx -g 'daemon off;'

## Build the image
* Tag it with <docker_id>/nginx:latest
* Push it to Docker Hub

## Verify the image works
* Start a container
* Expose port 80
* Use a browser

## Stop the container
docker container kill <CONTAINER NAME>
docker container ls
```

Solution:


**Dockerfile**

```## Base image (FROM)
FROM debian:buster-slim

## Run the following commands (RUN)
RUN apt-get update
RUN apt-get install -y nginx
EXPOSE 80
## Startup command (CMD)
CMD ["nginx", "-g", "daemon off;"]
```

========================================================

**Container lifecycle**


We already know how to run, stop, remove, attach, detach a container.

**Visit you container**

What if we want to debug or to check something inside the container without running the CMD commands inside the Dockerfile?

In that case we can use `$ docker container exec` like this:

1.First you need to run the container (in this case we run it in a detach mode):

```$ docker container run -d --name c1 nginx:latest 
ce3d222fb8b1724307f05c3956e6cee3a59092fd27bb4647706d1e021003682a
```

2. Do whatever we want using exec command like the couple examples below:

ex1:


```$ docker container exec c1 cat /etc/nginx/nginx.conf

user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

ex2:

```$ docker container exec -it c1 sh
# ls
bin   docker-entrypoint.d   home   media  proc	sbin  tmp
boot  docker-entrypoint.sh  lib    mnt	  root	srv   usr
dev   etc		    lib64  opt	  run	sys   var
# 
```

===========================================================

**Interacting with containers**

We know that in order to run an interactive program like shell or bash in our container we need to supply `-i` and `-t` flags (or `-it`).
Let's see what each flag exactly do:
1. run container with `-i` flag:

```$ docker container run -i alpine sh

ls
bin
dev
etc
home
lib
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
```

So, flag `-i` allows us to send commands to the shell inside container

2. run container with `-t` flag:

```$ docker container run -t alpine:latest
/ # ls
```

With `-t` flage we can see the terminal (or the prompts) but we cannot send commands.

3. run container with `-it` flag:

```$ docker container run -it alpine:latest
/ # ls
bin    etc    lib    mnt    proc   run    srv    tmp    var
dev    home   media  opt    root   sbin   sys    usr
/ # mkdir test
/ # ls
bin    etc    lib    mnt    proc   run    srv    test   usr
dev    home   media  opt    root   sbin   sys    tmp    var
/ # 
```

If we use both flags `-i` & `-t` (`-it`) together we can see the prompts and we can send commands to shell.

====================================================

**Stopping a container**


Apart from using ctrl+D or ctrl+C we have two ways to stop the container:


**Dockerfile**
```FROM alpine:latest

RUN apk update && apk add bash
COPY traptest.sh /

CMD ["/traptest.sh"]
```



**traptest.sh**
```#!/usr/bin/env bash
trap "echo Received SIGTERM > /dev/stderr" SIGTERM
trap "echo Received SIGINT  > /dev/stderr" SIGINT
trap "echo Received SIGHUP  > /dev/stderr" SIGHUP
trap "echo Received SIGQUIT > /dev/stderr" SIGQUIT
echo "Started with PID $$"

while :     # This is the same as "while true".
do
  sleep 0.1 # This script is not really doing anything.
  wait
done
```

```$ docker image build -t stop_demo .
...
$ 
```
1.using `stop`:

Window1:

```$ docker container run --name c2 stop_demo:latest
Started with PID 1
```
Window2:

```$ docker container stop c2
c2
```

Window1:
```$ docker container run --name c2 stop_demo:latest
Started with PID 1
Received SIGTERM
```

When we run `stop` command, `SIGTERM` signal will be sent to the shell and after 10 seconds default timeout limit the container will be exited.

```$ docker container ls -a
CONTAINER ID   IMAGE              COMMAND                  CREATED          STATUS                        PORTS     NAMES
43a483a68d0a   stop_demo:latest   "/traptest.sh"           3 minutes ago    Exited (137) 3 minutes ago              c2
```

2. using `kill` command:

Window1:

```$ docker container run --name c2 stop_demo:latest
Started with PID 1
```

Window2:
```$ docker container kill c2
c2
```
Window1:
```$ docker container run --name c2 stop_demo:latest
Started with PID 1
$
$ docker container ls -a
CONTAINER ID   IMAGE              COMMAND          CREATED          STATUS                        PORTS     NAMES
d1ea17c6bee7   stop_demo:latest   "/traptest.sh"   14 seconds ago   Exited (137) 7 seconds ago              c2
```

Common exit codes associated with docker containers are:

**Exit Code 0**: Absence of an attached foreground process
**Exit Code 1**: Indicates failure due to application error
**Exit Code 137**: Indicates failure as container received SIGKILL (Manual intervention or ‘oom-killer’ [OUT-OF-MEMORY])
**Exit Code 139**: Indicates failure as container received SIGSEGV
**Exit Code 143**: Indicates failure as container received SIGTERM

for more info:

[https://medium.com/better-programming/understanding-docker-container-exit-codes...](https://medium.com/better-programming/understanding-docker-container-exit-codes-5ee79a1d58f6)

==================================================================


