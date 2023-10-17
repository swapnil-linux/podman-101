# Practice: Create a Web Application Container

1. use podman search to search tomcat image

```
podman search tomcat
```

2\. use podman pull to download tomcat image from docker hub

```
[root@servera ~]# podman pull docker.io/tomcat
Trying to pull docker.io/tomcat...
Getting image source signatures
Copying blob d77915b4e630 done  
Copying blob a867dba77389 done  
Copying blob 27a2d52b526e done  
Copying blob 96b2c1e36db5 done  
Copying blob 5f37a0a41b6b done  
Copying blob 756975cb9c7e done  
Copying blob 0b0694ce0ae2 done  
Copying blob 0939c055fb79 done  
Copying blob c3d7917d545e done  
Copying blob 81a5f8099e05 done  
Copying config e0bd8b34b4 done  
Writing manifest to image destination
Storing signatures
e0bd8b34b4ea904874e55eae50e8987815030d140f9773a4b61759f4f85bf38d

```

3\. inspect the image to find the exposed ports

```
[root@servera ~]# podman inspect docker.io/tomcat|grep -A2 Ports
            "ExposedPorts": {
                "8080/tcp": {}
            },

```

4\. create a container publishing port 8080 of container to 38080 of your system

```
[root@servera ~]# podman run --name=web1 -d -p 38080:8080 docker.io/tomcat
32946afaa4538fbdd5f37495bc5145a76df6d6504545f36b717f1992ba400ca2

```

5\. verify the port forwarding using podman port, podman ps and iptables commands

```
[root@servera ~]# podman ps
CONTAINER ID  IMAGE                            COMMAND          CREATED        STATUS            PORTS                    NAMES
32946afaa453  docker.io/library/tomcat:latest  catalina.sh run  3 minutes ago  Up 3 minutes ago  0.0.0.0:38080->8080/tcp  web1

[root@servera ~]# podman port web1 
8080/tcp -> 0.0.0.0:38080
[root@servera ~]#

```

```
[root@servera ~]# iptables -t nat -L -n |grep 38080
CNI-HOSTPORT-SETMARK  tcp  --  10.88.0.15           0.0.0.0/0            tcp dpt:38080
CNI-HOSTPORT-SETMARK  tcp  --  127.0.0.1            0.0.0.0/0            tcp dpt:38080
DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:38080 to:10.88.0.15:8080
CNI-DN-93576c8c9886ba472d914  tcp  --  0.0.0.0/0            0.0.0.0/0            /* dnat name: "podman" id: "32946afaa4538fbdd5f37495bc5145a76df6d6504545f36b717f1992ba400ca2" */ multiport dports 38080
```

6\. Download a sample application in the container from [https://github.com/AKSarav/SampleWebApp/raw/master/dist/SampleWebApp.war](https://github.com/AKSarav/SampleWebApp/raw/master/dist/SampleWebApp.war)

```
[root@servera ~]# podman exec -it web1 bash
root@32946afaa453:/usr/local/tomcat#
```

```
root@32946afaa453:/usr/local/tomcat# cd webapps
root@32946afaa453:/usr/local/tomcat/webapps#
```

```
root@32946afaa453:/usr/local/tomcat/webapps# wget https://github.com/AKSarav/SampleWebApp/raw/master/dist/SampleWebApp.war

--2020-12-09 08:55:27--  https://github.com/AKSarav/SampleWebApp/raw/master/dist/SampleWebApp.war
Resolving github.com (github.com)... 13.237.44.5
Connecting to github.com (github.com)|13.237.44.5|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://raw.githubusercontent.com/AKSarav/SampleWebApp/master/dist/SampleWebApp.war [following]
--2020-12-09 08:55:27--  https://raw.githubusercontent.com/AKSarav/SampleWebApp/master/dist/SampleWebApp.war
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 151.101.28.133
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|151.101.28.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 8618 (8.4K) [application/octet-stream]
Saving to: ‘SampleWebApp.war’

SampleWebApp.war                100%[=======================================================>]   8.42K  --.-KB/s    in 0s      

2020-12-09 08:55:27 (60.4 MB/s) - ‘SampleWebApp.war’ saved [8618/8618]

root@32946afaa453:/usr/local/tomcat/webapps# ls
SampleWebApp.war

root@32946afaa453:/usr/local/tomcat/webapps# exit
[root@servera ~]# 


```

7\. access the application in your browser using your systems IP or hostname and the published port. for example http://yousysem:38080/SampleWebApp/welcome.jsp&#x20;

8\. Final clean up

```
[root@servera ~]# podman kill web1 
32946afaa4538fbdd5f37495bc5145a76df6d6504545f36b717f1992ba400ca2

[root@servera ~]# podman rm web1 
32946afaa4538fbdd5f37495bc5145a76df6d6504545f36b717f1992ba400ca2

[root@servera ~]# podman rmi docker.io/library/tomcat
Untagged: docker.io/library/tomcat:latest
Deleted: e0bd8b34b4ea904874e55eae50e8987815030d140f9773a4b61759f4f85bf38d
[root@servera ~]#
```

_**Congratulations! you successfully managed to create a web application container.**_
