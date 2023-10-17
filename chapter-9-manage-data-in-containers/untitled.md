# Data Volumes

Data volumes are used to manage data with podman. A data volume is a specially-designated directory within one or more containers that bypasses the Union File System. Data volumes provide several useful features for persistent or shared data:

* Volumes are initialised when a container is created. If the container’s parent image contains data at the specified mount point, that existing data is copied into the new volume upon volume initialisation.
* Data volumes can be shared and reused among containers.
* Changes to a data volume are made directly.
* Changes to a data volume will not be included when you update an image.
* Data volumes persist even if the container itself is deleted.

Data volumes are designed to persist data, independent of the container’s lifecycle. Docker therefore never automatically deletes volumes when you remove a container, nor will it “garbage collect” volumes that are no longer referenced by a container.

### Add a data volume

You can add a data volume to a container using the -v flag with the podman run command. You can use the -v multiple times to mount multiple data volumes.

```
podman run -it -v /opt/data:/data docker.io/alpine sh
```

```
[root@servera ~]# podman inspect kind_gates |grep  -A 13 Mounts
        "Mounts": [
            {
                "Type": "bind",
                "Name": "",
                "Source": "/opt/data",
                "Destination": "/data",
                "Driver": "",
                "Mode": "",
                "Options": [
                    "rbind"
                ],
                "RW": true,
                "Propagation": "rprivate"
            }
```

SELinux plays a very important role when mounting a host directory as a data volume. To secure host from applications running in containers, SELinux policies do not allow containers to access files and folders on host. To use a host directory in containers set `container_file_t` label on the directory. This can be done using `chcon` command before creating the container or, using the `:z` flag with the -`-volume` option in podman run command.

```
chcon -t container_file_t /opt/data
```

or

```
podman run -it -v /opt/data:/data:z docker.io/alpine sh
```









