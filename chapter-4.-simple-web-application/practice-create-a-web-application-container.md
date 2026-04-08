# Practice: Create a Web Application Container

1. use podman search to search tomcat image

```
podman search tomcat
```

2\. use podman pull to download tomcat image from docker hub

```
$ podman pull docker.io/tomcat
Trying to pull docker.io/library/tomcat:latest...
Getting image source signatures
Copying blob d77915b4e630 done   |
Copying blob a867dba77389 done   |
Copying blob 27a2d52b526e done   |
Copying blob 96b2c1e36db5 done   |
Copying blob 5f37a0a41b6b done   |
Copying config e0bd8b34b4 done   |
Writing manifest to image destination
e0bd8b34b4ea904874e55eae50e8987815030d140f9773a4b61759f4f85bf38d

```

3\. inspect the image to find the exposed ports

```
$ podman inspect docker.io/tomcat|grep -A2 Ports
            "ExposedPorts": {
                "8080/tcp": {}
            },

```

4\. create a container publishing port 8080 of container to 38080 of your system

```
$ podman run --name=web1 -d -p 38080:8080 docker.io/tomcat
32946afaa4538fbdd5f37495bc5145a76df6d6504545f36b717f1992ba400ca2

```

5\. verify the port forwarding using podman port and podman ps commands

```
$ podman ps
CONTAINER ID  IMAGE                            COMMAND          CREATED        STATUS            PORTS                    NAMES
32946afaa453  docker.io/library/tomcat:latest  catalina.sh run  3 minutes ago  Up 3 minutes ago  0.0.0.0:38080->8080/tcp  web1

$ podman port web1 
8080/tcp -> 0.0.0.0:38080

```

6\. Download a sample application in the container from [https://github.com/AKSarav/SampleWebApp/raw/master/dist/SampleWebApp.war](https://github.com/AKSarav/SampleWebApp/raw/master/dist/SampleWebApp.war)

```
$ podman exec -it web1 bash
root@32946afaa453:/usr/local/tomcat#
```

```
root@32946afaa453:/usr/local/tomcat# cd webapps
root@32946afaa453:/usr/local/tomcat/webapps#
```

```
root@32946afaa453:/usr/local/tomcat/webapps# wget https://github.com/AKSarav/SampleWebApp/raw/master/dist/SampleWebApp.war

root@32946afaa453:/usr/local/tomcat/webapps# ls
SampleWebApp.war

root@32946afaa453:/usr/local/tomcat/webapps# exit
```

7\. access the application in your browser using your systems IP or hostname and the published port. for example http://yoursystem:38080/SampleWebApp/welcome.jsp&#x20;

8\. Final clean up

```
$ podman kill web1 
32946afaa453

$ podman rm web1 
32946afaa453

$ podman rmi docker.io/library/tomcat
Untagged: docker.io/library/tomcat:latest
Deleted: e0bd8b34b4ea904874e55eae50e8987815030d140f9773a4b61759f4f85bf38d
```

_**Congratulations! you successfully managed to create a web application container.**_
