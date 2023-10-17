# podman run

Now that we have everything setup, it’s time to get our hands dirty. In this section, you are going to run an Alpine Linux container (a lightweight Linux distribution) on your system and get a taste of the podman run command.

```
[root@servera ~]# podman run docker.io/alpine echo hello world
Trying to pull docker.io/alpine...
Getting image source signatures
Copying blob 188c0c94c7c5 done  
Copying config d6e46aa247 done  
Writing manifest to image destination
Storing signatures
hello world
```

`podman run` does 3 things

1. pulls the images if it's not already available on the system
2. creates the namespaces
3. and launches the process in the namespace, in this case, its `echo hello world`

The container terminates if the primary process of the container, to get the list of running containers use `podman ps`

```
[root@servera ~]# podman ps
CONTAINER ID  IMAGE  COMMAND  CREATED  STATUS  PORTS  NAMES
```

and to get the list of all containers, running or exited use `podman ps -a`

```
[root@servera ~]# podman ps -a
CONTAINER ID  IMAGE                            COMMAND           CREATED        STATUS                    PORTS  NAMES
8adc741e3e28  docker.io/library/alpine:latest  echo hello world  6 minutes ago  Exited (0) 6 minutes ago         tender_wu
```

Let's understand the output of the command

* CONTAINER ID - A unique ID is assigned to each container. Containers can be referred to either using names or ID
* IMAGE - alpine image used to create this container
* COMMAND - the process that started while starting this container.&#x20;
* CREATED - when this container was created&#x20;
* STATUS - shows the current status of container, Exited in our case and so it was not listed with podman ps
* PORTS - Exposed ports of the application running in the container.&#x20;
* NAMES - A name given to container to be used to manage it using a name instead of CONTAINER ID

{% hint style="info" %}
NAMES are randomly generated by combining a list of adjectives like adoring, admiring, angry, tender... to a list of notable scientists and hackers.&#x20;

&#x20;[https://raw.githubusercontent.com/moby/moby/master/pkg/namesgenerator/names-generator.go](https://raw.githubusercontent.com/moby/moby/master/pkg/namesgenerator/names-generator.go)
{% endhint %}

To run a shell (Terminal) as a process in a container we need to run the process in and interactive way and allocate a pseudo-TTY

```
[root@servera ~]# podman run -it --name=myubuntu docker.io/ubuntu bash
Trying to pull docker.io/ubuntu...
Getting image source signatures
Copying blob da7391352a9b done  
Copying blob 2c2d948710f2 done  
Copying blob 14428a6d4bcd done  
Copying config f643c72bc2 done  
Writing manifest to image destination
Storing signatures
root@c378d88d23f9:/# 
```

* `-i, --interactive` Keep STDIN open even if not attached&#x20;
* `-t, --tty` Allocate a pseudo-TTY for container&#x20;
* `--name string` Assign a name to the container

running podman ps in another terminal will show running container

```
[root@servera ~]# podman ps
CONTAINER ID  IMAGE                            COMMAND  CREATED        STATUS            PORTS  NAMES
c378d88d23f9  docker.io/library/ubuntu:latest  bash     3 minutes ago  Up 3 minutes ago         myubuntu
```