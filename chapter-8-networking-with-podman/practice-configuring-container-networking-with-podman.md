# Practice: Configuring container networking with Podman

1. Explore the default podman network

```
$ podman network ls
NETWORK ID    NAME        DRIVER
2f259bab93aa  podman      bridge

$ podman network inspect podman
[
     {
          "name": "podman",
          "id": "2f259bab93aaaa17...",
          "driver": "bridge",
          "network_interface": "podman0",
          "created": "2025-01-15T10:00:00Z",
          "subnets": [
               {
                    "subnet": "10.88.0.0/16",
                    "gateway": "10.88.0.1"
               }
          ],
          "ipv6_enabled": false,
          "internal": false,
          "dns_enabled": false,
          "ipam_options": {
               "driver": "host-local"
          }
     }
]
```

{% hint style="info" %}
Notice that `dns_enabled` is `false` on the default network. DNS-based container name resolution only works on custom networks.
{% endhint %}

2\. Create another network using podman network create

```
$ podman network create mynetwork
mynetwork

$ podman network ls
NETWORK ID    NAME        DRIVER
2f259bab93aa  podman      bridge
a1b2c3d4e5f6  mynetwork   bridge
```

3\. Inspect the new network — notice DNS is enabled by default on custom networks

```
$ podman network inspect mynetwork
[
     {
          "name": "mynetwork",
          "id": "a1b2c3d4e5f6...",
          "driver": "bridge",
          "network_interface": "podman1",
          "subnets": [
               {
                    "subnet": "10.89.0.0/24",
                    "gateway": "10.89.0.1"
               }
          ],
          "dns_enabled": true,
          ...
     }
]
```

4\. Create an nginx container publishing all ports

```
podman run -dt --rm --publish-all docker.io/library/nginx
```

5\. Access the application

```
$ podman port -l
80/tcp -> 0.0.0.0:40684

$ curl http://localhost:40684
...Welcome to nginx!...
```

6\. Create an nginx container on the new network

```
podman run -dt --rm --name=nginx1 --net=mynetwork docker.io/library/nginx
```

7\. Create another alpine container on the same network, discover the IP of the first container and try accessing it

```
$ podman inspect -f "{{.NetworkSettings.Networks.mynetwork.IPAddress}}" nginx1
10.89.0.2

$ podman run -it --rm --net=mynetwork docker.io/alpine sh

/ # wget -qO- http://10.89.0.2
...Welcome to nginx!...
```

8\. Create another alpine container sharing the network namespace with nginx1 container

```
$ podman run -itd --name=alpine1 --net=container:nginx1 docker.io/alpine sh

$ podman ps --ns
CONTAINER ID  NAMES    PID   CGROUPNS  IPC         MNT         NET         PIDNS       USERNS      UTS
e57f50102324  alpine1  3220            4026532381  4026532379  4026532305  4026532382  4026531837  4026532380
0da418e07456  nginx1   2770            4026532377  4026532375  4026532305  4026532378  4026531837  4026532376

```

{% hint style="info" %}
Notice the NET namespace ID is the same for nginx1 and alpine1 containers
{% endhint %}

9\. Access the application from alpine1 container using localhost

```
$ podman exec -it alpine1 sh

/ # wget -qO- http://localhost
...Welcome to nginx!...
```

{% hint style="info" %}
Notice that the application is not running in this container but we are able to access it using localhost as both containers share the same network namespace
{% endhint %}

10\. Test rootless networking — login as a non-root user and create containers

```
$ podman run -dt --rm --name=nginx1 -p 8081:80 docker.io/library/nginx
$ podman run -dt --rm --name=nginx2 -p 8082:80 docker.io/library/nginx

$ curl http://localhost:8081
...Welcome to nginx!...

$ curl http://localhost:8082
...Welcome to nginx!...
```

11\. Final clean up

```
$ podman kill $(podman ps -q) 

$ podman system prune --all -f --volumes
```

_**Congratulations! you completed the exercise.**_
