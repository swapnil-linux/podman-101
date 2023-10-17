# Application Container

Great! So you have now looked at podman run, played with oci container and also got the hang of some terminologies. Armed with all this knowledge, you are now ready to get to the real stuff — deploying web applications with podman.

```
[root@servera ~]# podman run -d docker.io/swapnillinux/apache-php
Trying to pull docker.io/swapnillinux/apache-php...
Getting image source signatures
Copying blob 8ba884070f61 done  
Copying blob d642a70ab9a7 done  
Copying blob 688a607d5190 done  
Copying blob be0262a31194 done  
Copying config 26d33e5b4a done  
Writing manifest to image destination
Storing signatures
9e82b01f424f7c98974f86feb2b26954bb3dfc433899171911edaa4e94ba42a3
```

`-d, --detach` will run the container in background and print container ID

```
[root@servera ~]# podman inspect docker.io/swapnillinux/apache-php |grep -A3 Ports
            "ExposedPorts": {
                "443/tcp": {},
                "80/tcp": {}
            },
```

`podman inspect` displays the configuration of a container or image. Here we see some port information in the image. Inspecting container will show the IP address of the container.

```
[root@servera ~]# podman ps
CONTAINER ID  IMAGE                                     COMMAND      CREATED         STATUS             PORTS  NAMES
9e82b01f424f  docker.io/swapnillinux/apache-php:latest  /startup.sh  21 minutes ago  Up 21 minutes ago         goofy_hertz

[root@servera ~]# podman inspect goofy_hertz |grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "10.88.0.12",
```

As we have already created a web application container lets try to access it. We create a sample index.html and copy it to the containers /var/www/html folder using podman cp command&#x20;

```
[root@servera ~]# podman ps
CONTAINER ID  IMAGE                                     COMMAND      CREATED         STATUS             PORTS  NAMES
9e82b01f424f  docker.io/swapnillinux/apache-php:latest  /startup.sh  25 minutes ago  Up 25 minutes ago         goofy_hertz

[root@servera ~]# echo Hello > index.html

```

```
[root@servera ~]# podman cp --help
Copy files/folders between a container and the local filesystem

Description:
  Command copies the contents of SRC_PATH to the DEST_PATH.

  You can copy from the container's file system to the local machine or the reverse, from the local filesystem to the container. If "-" is specified for either the SRC_PATH or DEST_PATH, you can also stream a tar archive from STDIN or to STDOUT. The CONTAINER can be a running or stopped container.  The SRC_PATH or DEST_PATH can be a file or directory.


Usage:
  podman cp [flags] SRC_PATH DEST_PATH

Examples:
  [CONTAINER:]SRC_PATH [CONTAINER:]DEST_PATH

Flags:
      --extract   Extract the tar file into the destination directory.
      --pause     Pause the container while copying (default true)

[root@servera ~]# 
```

```
[root@servera ~]# podman cp index.html goofy_hertz:/var/www/html/index.html

[root@servera ~]# curl http://10.88.0.12
Hello
```

This container IP Address is only accessible from the host where it is running. To expose the container application outside the host we need to do port-forwarding. We will be discussing more about networking in later units. Now as we know our web server is using port 80 and 443 we can port-forward port 30080 of our host to port 80 of our container. Let's try it.

```
[root@servera ~]# podman run -d -p 30080:80 docker.io/swapnillinux/apache-php
d0cc914e886ad5385723b9174af3572d04da6d229d22a81bf3030a073ae17f84

[root@servera ~]# podman ps
CONTAINER ID  IMAGE                                     COMMAND      CREATED         STATUS             PORTS                  NAMES
d0cc914e886a  docker.io/swapnillinux/apache-php:latest  /startup.sh  4 seconds ago   Up 4 seconds ago   0.0.0.0:30080->80/tcp  crazy_shockley
9e82b01f424f  docker.io/swapnillinux/apache-php:latest  /startup.sh  31 minutes ago  Up 31 minutes ago                         goofy_hertz
[root@servera ~]#
```

`-p, --publish strings`    Publish a container's **port**, or a range of **port**s, to the host

Let's try to access it.

```
[root@servera ~]# echo Hello Podman > index2.html

[root@servera ~]# podman ps
CONTAINER ID  IMAGE                                     COMMAND      CREATED         STATUS             PORTS                  NAMES
d0cc914e886a  docker.io/swapnillinux/apache-php:latest  /startup.sh  2 minutes ago   Up 2 minutes ago   0.0.0.0:30080->80/tcp  crazy_shockley
9e82b01f424f  docker.io/swapnillinux/apache-php:latest  /startup.sh  34 minutes ago  Up 34 minutes ago                         goofy_hertz

[root@servera ~]# podman cp index2.html crazy_shockley:/var/www/html/index.html

[root@servera ~]# curl http://localhost:30080
Hello Podman

[root@servera ~]# iptables -t nat -L -n |grep 30080
CNI-HOSTPORT-SETMARK  tcp  --  10.88.0.13           0.0.0.0/0            tcp dpt:30080
CNI-HOSTPORT-SETMARK  tcp  --  127.0.0.1            0.0.0.0/0            tcp dpt:30080
DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:30080 to:10.88.0.13:80
CNI-DN-7731207a39b244780b697  tcp  --  0.0.0.0/0            0.0.0.0/0            /* dnat name: "podman" id: "d0cc914e886ad5385723b9174af3572d04da6d229d22a81bf3030a073ae17f84" */ multiport dports 30080
```

have a look at the iptables output, we will discuss this more in the networking chapter.&#x20;
