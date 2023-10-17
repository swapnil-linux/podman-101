# Practice: Configuring container networking with Podman

1. Explore the default podman network

```
[root@servera ~]# cat /etc/cni/net.d/87-podman-bridge.conflist 
{
    "cniVersion": "0.4.0",
    "name": "podman",
    "plugins": [
	{
            "type": "bridge",
            "bridge": "cni-podman0",
            "isGateway": true,
            "ipMasq": true,
            "ipam": {
		"type": "host-local",
		"routes": [
		    {
			"dst": "0.0.0.0/0"
		    }
		],
		"ranges": [
		    [
			{
			    "subnet": "10.88.0.0/16",
			    "gateway": "10.88.0.1"
			}
		    ]
		]
            }
	},
	{
            "type": "portmap",
            "capabilities": {
		"portMappings": true
            }
	},
	{
            "type": "firewall"
	}
    ]
}
[root@servera ~]#
```

```
[root@servera ~]# podman network ls
NAME     VERSION   PLUGINS
podman   0.4.0     bridge,portmap,firewall

[root@servera ~]# podman network inspect podman
```

```
[root@servera ~]# grep network /usr/share/containers/libpod.conf 
# Default CNI network for libpod.
# If multiple CNI network configs are present, libpod will use the network with
# precedence rules for selecting between multiple networks.
cni_default_network = "podman"
[root@servera ~]# 
```

2\. Create another network using podman network create

```
[root@servera ~]# podman network create 
/etc/cni/net.d/cni-podman1.conflist
```

```
[root@servera ~]# podman network ls
NAME          VERSION   PLUGINS
podman        0.4.0     bridge,portmap,firewall
cni-podman1   0.4.0     bridge,portmap,firewall
[root@servera ~]#
```

3\. Create an nginx container publishing all ports

```
podman run -dt --rm --publish-all quay.io/libpod/alpine_nginx
```

4\. Access the application

```
[root@servera ~]# podman port -l
80/tcp -> 0.0.0.0:40684


[root@servera ~]# curl http://localhost:40684
podman rulez
```

5\. create another nginx container of the new network

```
podman run -dt --rm --name=nginx1 --net=cni-podman1 quay.io/libpod/alpine_nginx
```

5\. create another alpine container on the same network, discover the IP of the first container and try accessing it from the alpine container

```
[root@servera ~]# podman inspect -f "{{.NetworkSettings.IPAddress}}" nginx1
10.89.0.3


[root@servera ~]# podman run -it --rm docker.io/alpine sh
Trying to pull docker.io/alpine...
Getting image source signatures
Copying blob 05e7bc50f07f done  
Copying config b14afc6dfb done  
Writing manifest to image destination
Storing signatures


/ # apk --update add curl
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/community/x86_64/APKINDEX.tar.gz
(1/4) Installing ca-certificates (20191127-r4)
(2/4) Installing nghttp2-libs (1.41.0-r0)
(3/4) Installing libcurl (7.69.1-r3)
(4/4) Installing curl (7.69.1-r3)
Executing busybox-1.31.1-r19.trigger
Executing ca-certificates-20191127-r4.trigger
OK: 7 MiB in 18 packages


/ # curl http://10.89.0.3
podman rulez
```

6\. create another alpine container sharing the network namespace with nginx1 container

```
[root@servera ~]# podman run -itd --name=alpine1 --net=container:nginx1 docker.io/alpine sh
e57f5010232449400ea99639bb5616176f8713ba7cc06aafc456e16038b174ae


[root@servera ~]# podman ps --ns
CONTAINER ID  NAMES                   PID   CGROUPNS  IPC         MNT         NET         PIDNS       USERNS      UTS
e57f50102324  alpine1                 3220            4026532381  4026532379  4026532305  4026532382  4026531837  4026532380
0da418e07456  nginx1                  2770            4026532377  4026532375  4026532305  4026532378  4026531837  4026532376

```

{% hint style="info" %}
notice the NET namespace id is same for nginx1 and alpine1 containers
{% endhint %}

7\. inspect the IP address of both container and try accessing the application from alpine1 container

```
[root@servera ~]# podman exec nginx1 ifconfig eth0
eth0      Link encap:Ethernet  HWaddr B2:0E:72:8D:E8:01  
          inet addr:10.89.0.3  Bcast:10.89.0.255  Mask:255.255.255.0
          inet6 addr: fe80::b00e:72ff:fe8d:e801/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:22 errors:0 dropped:0 overruns:0 frame:0
          TX packets:15 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:1705 (1.6 KiB)  TX bytes:1221 (1.1 KiB)
```

```
[root@servera ~]# podman exec alpine1 ifconfig eth0
eth0      Link encap:Ethernet  HWaddr B2:0E:72:8D:E8:01  
          inet addr:10.89.0.3  Bcast:10.89.0.255  Mask:255.255.255.0
          inet6 addr: fe80::b00e:72ff:fe8d:e801/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:22 errors:0 dropped:0 overruns:0 frame:0
          TX packets:15 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:1705 (1.6 KiB)  TX bytes:1221 (1.1 KiB)
```

```
[root@servera ~]# podman exec -it alpine1 sh

/ # apk --update add curl
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/community/x86_64/APKINDEX.tar.gz
(1/4) Installing ca-certificates (20191127-r4)
(2/4) Installing nghttp2-libs (1.41.0-r0)
(3/4) Installing libcurl (7.69.1-r3)
(4/4) Installing curl (7.69.1-r3)
Executing busybox-1.31.1-r19.trigger
Executing ca-certificates-20191127-r4.trigger
OK: 7 MiB in 18 packages


/ # ps -ef
PID   USER     TIME  COMMAND
    1 root      0:00 sh
   11 root      0:00 sh
   22 root      0:00 ps -ef
   
   
/ # curl http://localhost
podman rulez
```

{% hint style="info" %}
notice that the application is not running this contianer but we are able to access it using localhost as both containers share the same network namespace
{% endhint %}

8\. if you are running CentOS 7 or RHEL 7, enable user namespace by setting the value to greater than 0

```
[root@servera ~]# sysctl user.max_user_namespaces
user.max_user_namespaces = 0

[root@servera ~]# sysctl -w user.max_user_namespaces=1000
user.max_user_namespaces = 1000

[root@servera ~]# sysctl user.max_user_namespaces
user.max_user_namespaces = 1000

```

9\. Login as a non-root container and create 2 different nginx containers and inspect their IP Addresses

```
[swapnil@servera ~]$ podman run -dt --rm --name=nginx1  quay.io/libpod/alpine_nginx
2f58df4309ef1d713560f3f501705899ca8be5370d57aa663841226fd82b3e58


[swapnil@servera ~]$ podman run -dt --rm --name=nginx2  quay.io/libpod/alpine_nginx
de4b7a817b070f371b3eef82456d94323757c5e0c5afed8054c3244eaf9bd1af


[swapnil@servera ~]$ podman inspect -f "{{.NetworkSettings.IPAddress}}" nginx1

[swapnil@servera ~]$ podman inspect -f "{{.NetworkSettings.IPAddress}}" nginx2

[swapnil@servera ~]$ podman inspect nginx1 |grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "",


[swapnil@servera ~]$ podman exec nginx1 ifconfig eth0
ifconfig: eth0: error fetching interface information: Device not found
Error: non zero exit code: 1: OCI runtime error


[swapnil@servera ~]$ podman exec nginx1 ifconfig tap0
tap0      Link encap:Ethernet  HWaddr F2:8F:8D:38:D8:2F  
          inet addr:10.0.2.100  Bcast:10.0.2.255  Mask:255.255.255.0
          inet6 addr: fe80::f08f:8dff:fe38:d82f/64 Scope:Link
          UP BROADCAST RUNNING  MTU:65520  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:8 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:656 (656.0 B)


[swapnil@servera ~]$ podman exec nginx2 ifconfig tap0
tap0      Link encap:Ethernet  HWaddr AA:BA:BC:CB:38:5E  
          inet addr:10.0.2.100  Bcast:10.0.2.255  Mask:255.255.255.0
          inet6 addr: fe80::a8ba:bcff:fecb:385e/64 Scope:Link
          UP BROADCAST RUNNING  MTU:65520  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:8 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:656 (656.0 B)
```

```
[swapnil@servera ~]$ podman ps --ns
CONTAINER ID  NAMES   PID   CGROUPNS  IPC         MNT         NET         PIDNS       USERNS      UTS
de4b7a817b07  nginx2  3482            4026532531  4026532528  4026532459  4026532532  4026532226  4026532530
2f58df4309ef  nginx1  3434            4026532456  4026532453  4026532384  4026532457  4026532226  4026532455

```

10\. Final clean up

as non-root and root both

```
[swapnil@servera ~]$ podman kill $(podman ps -q) 
2f58df4309ef1d713560f3f501705899ca8be5370d57aa663841226fd82b3e58
de4b7a817b070f371b3eef82456d94323757c5e0c5afed8054c3244eaf9bd1af


[swapnil@servera ~]$ podman system prune --all -f --volumes
Deleted Pods
Deleted Containers
Deleted Volumes
Deleted Images
3ef70f7291f47dfe2b82931a993e16f5a44a0e7a68034c3e0e086d77f5829adc
[swapnil@servera ~]$ 
```

_**Congratulations! you completed the exercise.**_
