# Practice: Creating the first container

Step 1: Create a container using alpine image and run a process to print Hello World

```
$ podman run docker.io/alpine echo "Hello World"
Trying to pull docker.io/library/alpine:latest...
Getting image source signatures
Copying blob c6a83fedfae6 done   |
Copying config 1d34ffeaf1 done   |
Writing manifest to image destination
Hello World
```

Step 2: Create another container using RHEL9 based Universal Base Image (UBI), run it in interactive (i) mode to keep STDIN open and allocate a pseudo-TTY (t)

```
$ podman run -it registry.access.redhat.com/ubi9/ubi bash
Trying to pull registry.access.redhat.com/ubi9/ubi:latest...
Getting image source signatures
Checking if image destination supports signatures
Copying blob a3ed95caeb02 done   |
Copying blob 7aab3b38016c done   |
Copying config 5eb5554382 done   |
Writing manifest to image destination
Storing signatures
[root@a9ae02a3e71e /]# cat /etc/redhat-release 
Red Hat Enterprise Linux release 9.4 (Plow)
[root@a9ae02a3e71e /]#
```

Step 3: open another terminal and get the list of running containers

```
$ podman ps
CONTAINER ID  IMAGE                                       COMMAND  CREATED             STATUS                 PORTS  NAMES
a9ae02a3e71e  registry.access.redhat.com/ubi9/ubi:latest  bash     About a minute ago  Up About a minute ago         naughty_galois
```

Step 4: exit the shell of the running container and get the list of running containers and all containers

```
[root@a9ae02a3e71e /]# exit
exit

$ podman ps
CONTAINER ID  IMAGE  COMMAND  CREATED  STATUS  PORTS  NAMES

$ podman ps -a
CONTAINER ID  IMAGE                                       COMMAND           CREATED        STATUS                     PORTS  NAMES
a9ae02a3e71e  registry.access.redhat.com/ubi9/ubi:latest  bash              3 minutes ago  Exited (0) 51 seconds ago         naughty_galois
e06a14218e60  docker.io/library/alpine:latest             echo Hello World  6 minutes ago  Exited (0) 6 minutes ago          objective_colden
```

Step 5: start the container again and get the list of running containers

```
$ podman start a9ae02a3e71e
a9ae02a3e71e

$ podman ps
CONTAINER ID  IMAGE                                       COMMAND  CREATED        STATUS            PORTS  NAMES
a9ae02a3e71e  registry.access.redhat.com/ubi9/ubi:latest  bash     4 minutes ago  Up 2 seconds ago         naughty_galois

```

Step 6: get the list of images

```
$ podman images
REPOSITORY                            TAG      IMAGE ID       CREATED       SIZE
registry.access.redhat.com/ubi9/ubi   latest   5eb5554382e4   2 weeks ago   217 MB
docker.io/library/alpine              latest   1d34ffeaf190   4 weeks ago   7.79 MB
```

Step 7: stop all running containers and remove them

```
$ podman ps
CONTAINER ID  IMAGE                                       COMMAND  CREATED        STATUS                 PORTS  NAMES
a9ae02a3e71e  registry.access.redhat.com/ubi9/ubi:latest  bash     5 minutes ago  Up About a minute ago         naughty_galois

$ podman kill a9ae02a3e71e
a9ae02a3e71e

$ podman ps -a
CONTAINER ID  IMAGE                                       COMMAND           CREATED        STATUS                      PORTS  NAMES
a9ae02a3e71e  registry.access.redhat.com/ubi9/ubi:latest  bash              5 minutes ago  Exited (137) 4 seconds ago         naughty_galois
e06a14218e60  docker.io/library/alpine:latest             echo Hello World  8 minutes ago  Exited (0) 8 minutes ago           objective_colden

$ podman rm a9ae02a3e71e e06a14218e60
e06a14218e60
a9ae02a3e71e

$ podman ps -a
CONTAINER ID  IMAGE  COMMAND  CREATED  STATUS  PORTS  NAMES

```

_**Congratulations! you successfully managed a complete lifecycle of a container.**_
