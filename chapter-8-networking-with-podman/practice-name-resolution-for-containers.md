# Practice: Name resolution for containers

## dnsname plugin

The _dnsname_ plugin allows containers to resolve each other by name. The plugin adds each container's name to an instance of a dnsmasq server. The plugin is enabled through adding it to a network's CNI configuration. The containers will only be able to resolve each other if they are on the same CNI network.

{% hint style="info" %}
This plugin does not work with rootless containers.
{% endhint %}



1. Using your package manager, install the _dnsmasq_ package.

```
yum install dnsmasq -y
```

2\. install golang build environment

```
yum install go -y
```

3\. Build and install dnsname plugin for podman

```
RUN git clone https://github.com/containers/dnsname

cd dnsname/

make

make install PREFIX=/usr
```

4\. Create a new network using `podman network create`.

```
[root@servera myhttpd]# podman network create newnetwork 
/etc/cni/net.d/newnetwork.conflist
```

5\. you should be see dnsname plugin is now enabled for newnetworks

```
[root@servera myhttpd]# podman network ls
NAME         VERSION   PLUGINS
podman       0.4.0     bridge,portmap,firewall
newnetwork   0.4.0     bridge,portmap,firewall,dnsname
```

6\. review the network config file, to check the dnsname configuration

```
cat /etc/cni/net.d/newnetwork.conflist 
{
   "cniVersion": "0.4.0",
   "name": "newnetwork",
   "plugins": [
...
...
...
...
      {
         "type": "dnsname",
         "domainName": "dns.podman"
      }
   ]
}
```

7\. create a nginx container on newnetwork

```
[root@servera ~]# podman run -dt --name web --network newnetwork quay.io/libpod/alpine_nginx:latest
3471dd9b4c4e681508d0c6de9fd2093eb12cf2affb259925c7da69ec21d3c554
```

8\. try accessing nginx using name from another container

```
[root@servera ~]# podman run -it --rm --net newnetwork docker.io/alpine sh


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
/ # 
/ # 


/ # curl http://web
podman rulez
```

_**Congratulation!! that concludes the exercise.**_
