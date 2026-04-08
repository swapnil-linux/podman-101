# Default Networks

Podman uses **Netavark** as its default network stack for both rootful and rootless containers (since Podman 4.0). Netavark is a Rust-based network configuration tool purpose-built for containers. For DNS resolution between containers, Podman uses **Aardvark-DNS**, a companion authoritative DNS server.

For rootless containers, Podman uses **pasta** (from the passt project) as the default rootless network mode (since Podman 5.0), replacing the older slirp4netns.

### Networking and Podman pods <a href="#networking-and-podman-pods" id="networking-and-podman-pods"></a>

By definition, all containers in a Podman pod share the same network namespace. This fact means that they will have the same IP address, MAC addresses, and port mappings. You can conveniently communicate between containers in a pod by using localhost.

{% hint style="info" %}
We will discuss pods in a later chapter
{% endhint %}

### Rootful networking <a href="#rootfull-networking" id="rootfull-networking"></a>

When the Podman package is installed, a default network named `podman` is automatically available. The network configuration is stored in `/etc/containers/networks/` as JSON files. The default network settings (subnet, gateway, etc.) can be customized in `/etc/containers/containers.conf` under the `[network]` section.

To create a new network for rootful containers, use the `podman network` command:

```
$ sudo podman network create mynetwork
mynetwork
```

To display the currently configured networks, use the `podman network ls` command, and to remove a given network or networks, use `podman network rm`.

See the examples below to learn how to get various forms of communication working amongst rootful containers.

#### Communicate between a rootful container and the host <a href="#communicate-between-a-rootfull-container-and-the-host" id="communicate-between-a-rootfull-container-and-the-host"></a>

To communicate between the host container system and the `nginx` container, use the port mapping, which can be discovered with the `podman port` command:

```
$ sudo podman run -dt --rm --publish-all docker.io/library/nginx
34e3b5c686baa2298c37475864457e1b3a383c6d172266a8847257a33d790cc4
$ sudo podman port -l
80/tcp -> 0.0.0.0:40109
$ curl http://localhost:40109
...Welcome to nginx!...
```

#### Communicate between two rootful containers <a href="#communicate-between-two-rootfull-containers" id="communicate-between-two-rootfull-containers"></a>

Communicating between two rootful containers on the same network can be accomplished by using their IP addresses:

```
$ sudo podman run -dt --rm --name nginx docker.io/library/nginx
$ sudo podman inspect -f "{{.NetworkSettings.IPAddress}}" nginx
10.88.0.5
$ sudo podman run -it --rm docker.io/alpine sh
/ # wget -qO- http://10.88.0.5
...Welcome to nginx!...
```

#### DNS-based container name resolution <a href="#dns-based-name-resolution" id="dns-based-name-resolution"></a>

Containers on the same custom network (created with `podman network create`) can resolve each other by container name using the built-in **Aardvark-DNS** server. This works automatically — no additional plugins or configuration required:

```
$ sudo podman network create mynet
$ sudo podman run -dt --name web --network mynet docker.io/library/nginx
$ sudo podman run -it --rm --network mynet docker.io/alpine sh
/ # wget -qO- http://web
...Welcome to nginx!...
```

{% hint style="info" %}
DNS name resolution only works on custom (non-default) networks. Containers on the default `podman` network must use IP addresses to communicate.
{% endhint %}

#### Communicate between two rootful containers in a pod <a href="#communicate-between-two-rootfull-containers-in-a-pod" id="communicate-between-two-rootfull-containers-in-a-pod"></a>

Communicating between two containers in a pod is probably the simplest of all the methods. Because they are in the same network namespace, localhost can be used:

```
$ sudo podman run -dt --pod new:mypod --rm --name nginx docker.io/library/nginx
$ sudo podman run --pod mypod -it --rm docker.io/alpine sh
/ # wget -qO- http://localhost
...Welcome to nginx!...
```

### Rootless networking <a href="#rootless-networking" id="rootless-networking"></a>

Rootless containers use **pasta** (since Podman 5.0) for network connectivity. Pasta creates a network namespace with full connectivity by forwarding traffic between the container and the host network using a TAP interface. This provides a more seamless networking experience compared to the older slirp4netns approach.

Key characteristics of rootless networking with pasta:
- Containers get their own network namespace with a TAP device
- Port forwarding works via `-p` flag just like rootful containers
- DNS resolution works using Aardvark-DNS on custom networks
- Containers in a pod share the same network namespace and can communicate via localhost

To communicate amongst two or more rootless containers, there are two choices:

1. The easiest would be to put all of the containers into a singular pod. These containers can then communicate using localhost. Another benefit is that no ports need to be opened so that the containers can communicate with each other directly.
2. A second option is to create a custom network with `podman network create` and place containers on it, allowing DNS-based name resolution between containers.
