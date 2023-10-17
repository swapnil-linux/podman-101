# Practice: Creating the first container

Step 1: Create a container using alpine image and run a process to print Hello World

```
[root@servera ~]# podman run docker.io/alpine echo "Hello World"
Trying to pull docker.io/library/alpine...
Getting image source signatures
Copying blob 188c0c94c7c5 done  
Copying config d6e46aa247 done  
Writing manifest to image destination
Storing signatures
Hello World
```

Step 2: Create another container using RHEL7 based Universal Base Image (UBI), run it in interactive (i) mode to keep STDIN open and allocate a pseudo-TTY (t)

```
[root@servera ~]# podman run -it registry.access.redhat.com/ubi7/ubi bash
Trying to pull registry.access.redhat.com/ubi7/ubi...
Getting image source signatures
Copying blob 2b63304f1869 done  
Copying blob d518f7038c56 done  
Copying config cfd6d0c09e done  
Writing manifest to image destination
Storing signatures
[root@a9ae02a3e71e /]# cat /etc/redhat-release 
Red Hat Enterprise Linux Server release 7.9 (Maipo)
[root@a9ae02a3e71e /]#
```

Step 3: open another terminal and get the list of running containers

```
[root@servera ~]# podman ps
CONTAINER ID  IMAGE                                       COMMAND  CREATED             STATUS                 PORTS  NAMES
a9ae02a3e71e  registry.access.redhat.com/ubi7/ubi:latest  bash     About a minute ago  Up About a minute ago         naughty_galois
```

Step 4: exit the shell of the running container and get the list of running containers and all containers

```
[root@a9ae02a3e71e /]# exit
exit

[root@servera ~]# podman ps
CONTAINER ID  IMAGE  COMMAND  CREATED  STATUS  PORTS  NAMES

[root@servera ~]# podman ps -a
CONTAINER ID  IMAGE                                       COMMAND           CREATED        STATUS                     PORTS  NAMES
a9ae02a3e71e  registry.access.redhat.com/ubi7/ubi:latest  bash              3 minutes ago  Exited (0) 51 seconds ago         naughty_galois
e06a14218e60  docker.io/library/alpine:latest             echo Hello World  6 minutes ago  Exited (0) 6 minutes ago          objective_colden
[root@servera ~]#
```

Step 5: start the container again and get the list of running containers

```
[root@servera ~]# podman start a9ae02a3e71e
a9ae02a3e71e54d70d73800971e35afc8bea327d6282d672239cf5dd654173ce

[root@servera ~]# podman ps
CONTAINER ID  IMAGE                                       COMMAND  CREATED        STATUS            PORTS  NAMES
a9ae02a3e71e  registry.access.redhat.com/ubi7/ubi:latest  bash     4 minutes ago  Up 2 seconds ago         naughty_galois

```

Step 6: get the list of images

```
[root@servera ~]# podman images
REPOSITORY                            TAG      IMAGE ID       CREATED       SIZE
registry.access.redhat.com/ubi7/ubi   latest   cfd6d0c09e95   4 weeks ago   216 MB
docker.io/library/alpine              latest   d6e46aa2470d   6 weeks ago   5.85 MB
```

Step 7: stop all running containers and remove them

```
[root@servera ~]# podman ps
CONTAINER ID  IMAGE                                       COMMAND  CREATED        STATUS                 PORTS  NAMES
a9ae02a3e71e  registry.access.redhat.com/ubi7/ubi:latest  bash     5 minutes ago  Up About a minute ago         naughty_galois

[root@servera ~]# podman kill a9ae02a3e71e
a9ae02a3e71e54d70d73800971e35afc8bea327d6282d672239cf5dd654173ce

[root@servera ~]# podman ps -a
CONTAINER ID  IMAGE                                       COMMAND           CREATED        STATUS                      PORTS  NAMES
a9ae02a3e71e  registry.access.redhat.com/ubi7/ubi:latest  bash              5 minutes ago  Exited (137) 4 seconds ago         naughty_galois
e06a14218e60  docker.io/library/alpine:latest             echo Hello World  8 minutes ago  Exited (0) 8 minutes ago           objective_colden

[root@servera ~]# podman rm a9ae02a3e71e e06a14218e60
e06a14218e60dbc5ca433da1f9cced7ee9e44edebc9fad19e9fef31e1c161548
a9ae02a3e71e54d70d73800971e35afc8bea327d6282d672239cf5dd654173ce

[root@servera ~]# podman ps -a
CONTAINER ID  IMAGE  COMMAND  CREATED  STATUS  PORTS  NAMES
[root@servera ~]# 

```

_**Congratulations! you successfully managed a complete lifecycle of a container.**_
