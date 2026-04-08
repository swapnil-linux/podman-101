# Committing a Container

Podman allows you to save your containers as images. This is one way of converting and committing your container changes to a new image. `podman commit` command creates a new image from the container's changes. The new image will contain the contents of the container filesystem, excluding any data volumes.

```
$ podman run -it docker.io/alpine sh
Trying to pull docker.io/library/alpine:latest...
Getting image source signatures
Copying blob c6a83fedfae6 done   |
Copying config 1d34ffeaf1 done   |
Writing manifest to image destination

/ # echo hello world > /opt/test.txt

/ # echo Welcome >> /etc/issue 

/ # rm /etc/motd

/ # exit

$ podman ps -a
CONTAINER ID  IMAGE                            COMMAND  CREATED        STATUS                    PORTS  NAMES
32b4d6bf4160  docker.io/library/alpine:latest  sh       2 minutes ago  Exited (0) 2 seconds ago         laughing_curie

$ podman diff laughing_curie 
C /etc
C /etc/issue
D /etc/motd
C /opt
A /opt/test.txt
C /root
A /root/.ash_history

```

`podman diff` helps inspect changes on container's file systems

* A - New file/folder added
* C - existing file/folder changed
* D - existing file/folder deleted

podman commit will create a new image based on the changed container

```
$ podman commit laughing_curie alpine-new
Getting image source signatures
Copying blob ace0eda3e3be skipped: already exists  
Copying blob 7611facab743 done   |
Copying config f5890e7e9a done   |
Writing manifest to image destination
Storing signatures
f5890e7e9a3c8d645dda4a141469a561ef45a07b3f2a5dac1e5f6152d6000599


$ podman images
REPOSITORY                 TAG      IMAGE ID       CREATED         SIZE
localhost/alpine-new       latest   f5890e7e9a3c   9 seconds ago   7.79 MB
docker.io/library/alpine   latest   1d34ffeaf190   4 weeks ago     7.79 MB

```

While the podman commit is a convenient way of extending an existing image, you should prefer the use of a Containerfile (or Dockerfile) and podman build for generating images.
