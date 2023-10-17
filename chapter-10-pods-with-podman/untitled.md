# What is a Pod?

The Pod concept was introduced by Kubernetes. The shared context of a Pod is a set of Linux namespaces, cgroups, and potentially other facets of isolation - the same things that isolate a container. Within a Pod's context, the individual applications may have further sub-isolations applied.

In terms of container concepts, a Pod is similar to a group of containers with shared namespaces and shared filesystem volumes.

![](<../.gitbook/assets/image (2).png>)

**Every Podman pod includes an “infra” container.** This container does nothing, but go to sleep. Its purpose is to hold the namespaces associated with the pod and allow podman to connect other containers to the pod.  This allows you to start and stop containers within the POD and the pod will stay running, where as if the primary container controlled the pod, this would not be possible. The default infra container is based on the k8s.gcr.io/pause image,  Unless you explicitly say otherwise, all pods will have container based on the default image.

Most of the attributes that make up the Pod are actually assigned to the “infra” container.  Port bindings, cgroup-parent values, and kernel namespaces are all assigned to the “infra” container. This is critical to understand, because once the pod is created these attributes are assigned to the “infra” container and cannot be changed. For example, if you create a pod and then later decide you want to add a container that binds new ports, Podman will not be able to do this.  You would need to recreate the pod with the additional port bindings before adding the new container.

In the above diagram, notice the box above each container, **conmon, this is the container monitor.  It is a small C Program that’s job is to watch the primary process of the container,** and if the container dies, save the exit code.  It also holds open the tty of the container, so that it can be attached to later. This is what allows podman to run in detached mode (backgrounded), so podman can exit but conmon continues to run.  Each container has its own instance of conmon.

The traditional way to create a pod with Podman is using the `podman pod create` command.

```
[root@servera ~]# podman pod create --help
Create a new empty pod

Description:
  After creating the pod, the pod ID is printed to stdout.

  You can then start it at any time with the  podman pod start <pod_id> command. The pod will be created with the initial state 'created'.

Usage:
  podman pod create [flags]

Flags:
      --cgroup-parent string   Set parent cgroup for the pod
      --hostname string        Set a hostname to the pod
      --infra                  Create an infra container associated with the pod to share namespaces with (default true)
      --infra-command string   The command to run on the infra container when the pod is started (default "/pause")
      --infra-image string     The image of the infra container to associate with the pod (default "k8s.gcr.io/pause:3.1")
  -l, --label strings          Set metadata on pod (default [])
      --label-file strings     Read in a line delimited file of labels
  -n, --name string            Assign a name to the pod
      --pod-id-file string     Write the pod ID to the file
  -p, --publish strings        Publish a container's port, or a range of ports, to the host (default [])
      --share string           A comma delimited list of kernel namespaces the pod will share (default "cgroup,ipc,net,uts")

```

> ```
> [root@servera ~]# podman pod create
> 70401d508418ef5d9a3631fa2d4376ab6fa86526f6ec2c04eb2e599ffbf9cc09
>
>
> [root@servera ~]# podman pod ps
> POD ID         NAME            STATUS    CREATED          # OF CONTAINERS   INFRA ID
> 70401d508418   jolly_mestorf   Created   10 seconds ago   1                 0e02398d239d
> ```

Note that the container has a single container in it.  The container is the “infra” command. We can further observe this using the `podman ps` command by passing the command line switch --pod.

```
[root@servera ~]# podman ps -a --pod
CONTAINER ID  IMAGE                 COMMAND  CREATED             STATUS   PORTS  NAMES               POD
0e02398d239d  k8s.gcr.io/pause:3.1           About a minute ago  Created         70401d508418-infra  70401d508418
```

### Add a container to a pod

You can add a container to a pod using the --pod option in the `podman create` and `podman run` commands.  For example, here we add a container running `top` to the newly created jolly\_mestorf pod. Notice the use of --pod.

```
[root@servera ~]# podman run -dt --pod jolly_mestorf docker.io/alpine top
Trying to pull docker.io/alpine...
Getting image source signatures
Copying blob 05e7bc50f07f done  
Copying config b14afc6dfb done  
Writing manifest to image destination
Storing signatures
92cf1d5a4fdd10d9f0c0043d3636e48d3122a47c9e815be7203df69d1a414411


[root@servera ~]# podman pod ps
POD ID         NAME            STATUS    CREATED         # OF CONTAINERS   INFRA ID
70401d508418   jolly_mestorf   Running   5 minutes ago   2                 0e02398d239d
```

And now two containers exist in our pod. Looking at the list of containers, we also see each container and their respective pod assignment.

```
[root@servera ~]# podman ps -a --pod
CONTAINER ID  IMAGE                            COMMAND  CREATED         STATUS             PORTS  NAMES               POD
92cf1d5a4fdd  docker.io/library/alpine:latest  top      18 seconds ago  Up 18 seconds ago         adoring_volhard     70401d508418
0e02398d239d  k8s.gcr.io/pause:3.1                      5 minutes ago   Up 18 seconds ago         70401d508418-infra  70401d508418
```

### Shortcut to create pods

A new feature recently added the ability to create pods via the `podman run` command. One upside to creating a pod with this approach is that the normal port bindings declared for the container will be assigned automatically to the “infra” container. However, if you need to specify more granular options for pod creation like kernel namespaces or different “infra” container image usage, you still need to create the pod manually as was first described. Nevertheless, for relatively basic pod creations, the shortcut is handy.

To create a new pod with your new container, you simply pass `--pod: new:<name>`. The use of `new:` indicates to Podman that you want to create a new pod rather than attempt to assign the container to an existing pod.

To create a nginx container within a pod and expose port 80 from the container to port 32597 on the host, you would:

```
[root@servera ~]# podman run -dt --pod new:nginx -p 32597:80 quay.io/libpod/alpine_nginx:latest
Trying to pull quay.io/libpod/alpine_nginx:latest...
Getting image source signatures
Copying blob a3ed95caeb02 done  
Copying blob a3ed95caeb02 skipped: already exists  
Copying blob ee08fa06e364 done  
Copying blob a3ed95caeb02 done  
Copying blob a3ed95caeb02 done  
Copying blob 7ecd51378d2e done  
Copying blob 4fe2ade4980c done  
Writing manifest to image destination
Storing signatures
b775f06b6aa89c3f7f6b571c43cf43cae2998dcb729e4612ee37049fd5feb356
[root@servera ~]# 
```

And here is what it looks like when listing containers:

```
[root@servera ~]# podman pod ps
POD ID         NAME            STATUS    CREATED          # OF CONTAINERS   INFRA ID
9ad533c54fc7   nginx           Running   2 minutes ago    2                 0587f29da798
70401d508418   jolly_mestorf   Running   13 minutes ago   2                 0e02398d239d


[root@servera ~]# podman ps --all --pod 
CONTAINER ID  IMAGE                               COMMAND               CREATED         STATUS            PORTS                  NAMES               POD
b775f06b6aa8  quay.io/libpod/alpine_nginx:latest  nginx -g daemon o...  2 minutes ago   Up 2 minutes ago  0.0.0.0:32597->80/tcp  determined_lamarr   9ad533c54fc7
0587f29da798  k8s.gcr.io/pause:3.1                                      2 minutes ago   Up 2 minutes ago  0.0.0.0:32597->80/tcp  9ad533c54fc7-infra  9ad533c54fc7

```





