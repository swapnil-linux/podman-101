# Images and layers

A Docker image is built up from a series of layers. Each layer represents an instruction in the image’s Dockerfile. Each layer is read-only. Consider the following Dockerfile:

```
FROM ubuntu:20.04
COPY . /app
RUN make /app
CMD ["python","/app/app.py"]
```

This Dockerfile contains four instructions, each of which creates a layer. The FROM statement starts out by creating a layer from the ubuntu:20.04 image. The COPY command adds some files from your Docker client’s current directory. The RUN command builds your application using the make command. Finally, the last layer specifies what command to run within the container.&#x20;

Each layer is only a set of differences from the layer before it. The layers are stacked on top of each other. When you create a new container, you add a new writable layer on top of the underlying layers. This layer is often called the “container layer”. All changes made to the running container, such as writing new files, modifying existing files, and deleting files, are written to this thin writable container layer.

### Container and layers&#x20;

The major difference between a container and an image is the top writable layer. All writes to the container that adds new or modifies existing data are stored in this writable layer. When the container is deleted, the writable layer is also deleted. The underlying image remains unchanged. Because each container has its own writable container layer, and all changes are stored in this container layer, multiple containers can share access to the same underlying image and yet have their own data state.

### Container size on disk

To view the approximate size of a running container, you can use the `podman ps -s` command, you will see two different columns relate to size.

```
[root@servera ~]# podman ps -s
CONTAINER ID  IMAGE                            COMMAND          CREATED         STATUS             PORTS  NAMES              SIZE
271fda1ba4f1  docker.io/library/alpine:latest  sh               4 seconds ago   Up 4 seconds ago          flamboyant_diffie  0B (virtual 5.57MB)
5b6cfbe91757  docker.io/library/tomcat:latest  catalina.sh run  15 seconds ago  Up 14 seconds ago         zealous_gould      37.6kB (virtual 649MB)
[root@servera ~]# 

```

**size**: the amount of data (on disk) that is used for the writable layer of each container

**virtual** **size**: the amount of data used for the read-only image data used by the container. Multiple containers may share some or all read-only image data. Two containers started from the same image share 100% of the read-only data, while two containers with different images which have layers in common share those common layers. Therefore, you can’t just total the virtual sizes. This will over-estimate the total disk usage by a potentially non-trivial amount.

The total disk space used by all of the running containers on disk is some combination of each container’s size and the virtual size values. If multiple containers have exactly the same virtual size, they are likely started from the same exact image.

### The copy-on-write (CoW) strategy

Copy-on-write is a strategy of sharing and copying files for maximum efficiency. If a file or directory exists in a lower layer within the image, and another layer (including the writable layer) needs read access to it, it just uses the existing file. The first time another layer needs to modify the file (when building the image or running the container), the file is copied into that layer and modified. This minimizes I/O and the size of each of the subsequent layers.

### Sharing promotes smaller images

When you use podman pull to pull down an image from a repository, or when you create a container from an image that does not yet exist locally, each layer is pulled down separately, and stored in the local storage area, which is usually`/var/lib/containers/storage`on Linux hosts. You can see these layers being pulled in this example:

```
[root@servera ~]# podman pull docker.io/mysql:5.7
Trying to pull docker.io/mysql:5.7...
Getting image source signatures
Copying blob 852e50cd189d [============>-------------------------] 8.8MiB / 25.8MiB
Copying blob 5cdd802543a3 done  
Copying blob 938c64119969 [=========>----------------------------] 3.3MiB / 12.8MiB
Copying blob b79b040de953 done  
Copying blob a43f41a44c48 done  
Copying blob 29969ddb0ffb done  
```

Each of these layers is stored in its own directory inside the host’s local storage area. To examine the layers on the filesystem, list the contents of `/var/lib/containers/storage/overlay-layers`. podman info shows the storage driver and the local storage area path.&#x20;

```
[root@servera ~]# podman info
...
...
...
...
store:
  ConfigFile: /etc/containers/storage.conf
  ContainerStore:
    number: 2
  GraphDriverName: overlay
  GraphOptions: {}
  GraphRoot: /var/lib/containers/storage
  GraphStatus:
    Backing Filesystem: xfs
    Native Overlay Diff: "true"
    Supports d_type: "true"
    Using metacopy: "false"
  ImageStore:
    number: 2
  RunRoot: /var/run/containers/storage
  VolumePath: /var/lib/containers/storage/volumes

```
