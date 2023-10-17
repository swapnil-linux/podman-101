# Default Networks

Podman uses two different means for its networking stack, depending on whether the container is _rootless_ or _rootfull_. When _rootfull_, defined as being run by the root (or equivalent) user, Podman primarily relies on the [`containernetworking` plugins](https://github.com/containernetworking/plugins) project. When rootless, defined as being run by a regular user, Podman uses the [`slirp4netns`](https://github.com/rootless-containers/slirp4netns) project.

### Networking and Podman pods <a href="#networking-and-podman-pods" id="networking-and-podman-pods"></a>

By definition, all containers in a Podman pod share the same network namespace. This fact means that they will have the same IP address, MAC addresses, and port mappings. You can conveniently communicate between containers in a pod by using localhost.

{% hint style="info" %}
We will discuss pods in a later chapter
{% endhint %}

### Rootfull networking <a href="#rootfull-networking" id="rootfull-networking"></a>

When the Podman package is installed, a default network configuration is commonly installed into `/etc/cni/net.d` as [`87-podman-bridge.conflist`](https://github.com/containers/libpod/blob/master/cni/87-podman-bridge.conflist). In addition, the default network name is defined in `/usr/share/containers/libpod.conf` with the key `cni_default_network`. In an unaltered `libpod.conf`, the default network name will be that of the default configuration provided by the package. You can copy the `libpod.conf` file to `/etc/containers/libpod.conf` if you want to alter any of its default settings.

To create a new network for rootfull containers, use the `podman network` command. You can customize the network with the command flags or by editing it by hand afterward. The `podman network create` command provides you with the newly created network configuration file path:

```
$ sudo podman network create
/etc/cni/net.d/cni-podman4.conflist
```

To display the currently configured networks, use the `podman network ls` command, and to remove a given network or networks, use `podman network rm`.

{% hint style="info" %}
All `podman network` commands are for rootfull containers only.
{% endhint %}

See the examples below to learn how to get various forms of communication working amongst rootfull containers. As with many networking topics, there are multiple ways to accomplish a given result based on restrictions and needs. These examples reflect the simplest of those ways.

#### Communicate between a rootfull container and the host <a href="#communicate-between-a-rootfull-container-and-the-host" id="communicate-between-a-rootfull-container-and-the-host"></a>

To communicate between the host container system and the `nginx` container, use the port mapping, which can be discovered with the `podman port` command:

```
# podman run -dt --rm --publish-all quay.io/libpod/alpine_nginx
34e3b5c686baa2298c37475864457e1b3a383c6d172266a8847257a33d790cc4
# podman port -l
80/tcp -> 0.0.0.0:40109
# curl http://localhost:40109
podman rulez
```

#### Communicate between two rootfull containers <a href="#communicate-between-two-rootfull-containers" id="communicate-between-two-rootfull-containers"></a>

Communicating between two rootfull containers on the same network can be accomplished by using their IP addresses:

```
# podman run -dt --rm --name nginx quay.io/libpod/alpine_nginx
2e383ec1ca1873c3665e975bbaf40f98020a288534f686ce08fb243d321a653c
# sudo podman inspect -f "{{.NetworkSettings.IPAddress}}" nginx
10.88.6.9
# podman run -it --rm alpine_nginx /bin/sh
/ # curl http://10.88.6.9
podman rulez
```

#### Communicate between two rootfull containers in a pod <a href="#communicate-between-two-rootfull-containers-in-a-pod" id="communicate-between-two-rootfull-containers-in-a-pod"></a>

Communicating between two containers in a pod is probably the simplest of all the methods. Because they are in the same network namespace, localhost can be used:

```
[root@localhost libpod]# podman run -dt --pod new:mypod --rm --name nginx quay.io/libpod/alpine_nginx
b72432387f603cd4da14348537cfad6b85424ee5dbc1e3af79a527b01c2e59a6
[root@localhost libpod]# podman run --pod mypod -it --rm alpine_nginx /bin/sh
/ # curl http://localhost
podman rulez
```

### Rootless networking <a href="#rootless-networking" id="rootless-networking"></a>

When using Podman as a rootless user, the network setup is automatic. _Technically, the container itself does not have an IP address, because without root privileges, network device association cannot be achieved._ Moreover, pinging from a rootless container does not work because it lacks the CAP\_NET\_RAW security capability that the `ping` command requires. If you want to ping from within a rootless container, you can allow users to send ICMP packets using this `sysctl` command:

```
# sysctl -w "net.ipv4.ping_group_range=0 2000000" 
```

This action would allow any process within these groups to send `ping` packets.

To communicate amongst two or more rootless containers, there are two choices.&#x20;

1. The easiest would be to put all of the containers into a singular pod. These containers can then communicate using localhost. Another benefit is that no ports need to be opened so that the containers can communicate with each other directly.
2. A second option is to use a port mapping technique to map ports to containers and then use those ports to direct traffic to specific containers.

#### &#x20;<a href="#communicate-between-a-rootless-container-and-the-host" id="communicate-between-a-rootless-container-and-the-host"></a>
