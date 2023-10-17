# Practice: Container to Image

1: Create an alpine container and install htop

```
[root@servera ~]# podman run -it --name=testcommit docker.io/alpine sh
Trying to pull docker.io/alpine...
Getting image source signatures
Copying blob 188c0c94c7c5 done  
Copying config d6e46aa247 done  
Writing manifest to image destination
Storing signatures


/ # htop
sh: htop: not found


/ # apk update
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/community/x86_64/APKINDEX.tar.gz
v3.12.1-82-g0e1cfdcae4 [http://dl-cdn.alpinelinux.org/alpine/v3.12/main]
v3.12.1-83-ge8061d3671 [http://dl-cdn.alpinelinux.org/alpine/v3.12/community]
OK: 12747 distinct packages available


/ # apk add htop
(1/3) Installing ncurses-terminfo-base (6.2_p20200523-r0)
(2/3) Installing ncurses-libs (6.2_p20200523-r0)
(3/3) Installing htop (2.2.0-r0)
Executing busybox-1.31.1-r19.trigger
OK: 6 MiB in 17 packages
/ # 


```

2\. Test running htop, q to exit htop and exit the container.

```
/ # htop
/ # exit
```

3\. Inspect the changes done in the container layer using podman diff command

```
[root@servera ~]# podman diff testcommit
C /etc
C /etc/apk
C /etc/apk/world
A /etc/terminfo
A /etc/terminfo/a
..
..
..
.. 
```

4\. Create a new image named alpine/htop from a container's changes using podman commit command. Remove the testcommit container after saving it as image.

```
[root@servera ~]# podman commit testcommit alpine/htop
Getting image source signatures
Copying blob ace0eda3e3be skipped: already exists  
Copying blob 02a390e9ccf0 done  
Copying config afd869d361 done  
Writing manifest to image destination
Storing signatures
afd869d36178fdb6a5d4ac9958ccc932938d2f3d30ab3165c258bb75d8a1ee97

[root@servera ~]# podman images
REPOSITORY                 TAG      IMAGE ID       CREATED              SIZE
localhost/alpine/htop      latest   afd869d36178   About a minute ago   8.47 MB
docker.io/library/alpine   latest   d6e46aa2470d   7 weeks ago          5.85 MB
[root@servera ~]# 

[root@servera ~]# podman rm testcommit 
7af130de9d47e742205a6c3bf414459398bbce225fa2256d0ea96817e6c9f2f9
[root@servera ~]#
```

5\. Test the new image by creating a temporary container, without creating the PID namespace and starting htop process as a UID 1234

```
podman run --rm -it -u 1234 --pid=host alpine/htop htop
```

6\. Save and image as tar file using podman save command. Delete the image and load it back using podman load command.

```
[root@servera ~]# podman save alpine/htop > alpine_htop.tar

[root@servera ~]# tar tvf alpine_htop.tar|head
-r--r--r-- 0/0         5843456 1970-01-01 00:00 ace0eda3e3be35a979cec764a3321b4c7d0b9e4bb3094d20d3ff6782961a8d54.tar
-r--r--r-- 0/0         2625024 1970-01-01 00:00 02a390e9ccf0f6ab23ff00a9f8957f983f5e9892fb5f55f120e1bcdb68ec3cdf.tar
-r--r--r-- 0/0             782 1970-01-01 00:00 afd869d36178fdb6a5d4ac9958ccc932938d2f3d30ab3165c258bb75d8a1ee97.json
l--------- 0/0               0 1970-01-01 00:00 ace0eda3e3be35a979cec764a3321b4c7d0b9e4bb3094d20d3ff6782961a8d54/layer.tar -> ../ace0eda3e3be35a979cec764a3321b4c7d0b9e4bb3094d20d3ff6782961a8d54.tar
-r--r--r-- 0/0               3 1970-01-01 00:00 ace0eda3e3be35a979cec764a3321b4c7d0b9e4bb3094d20d3ff6782961a8d54/VERSION
-r--r--r-- 0/0              73 1970-01-01 00:00 ace0eda3e3be35a979cec764a3321b4c7d0b9e4bb3094d20d3ff6782961a8d54/json
l--------- 0/0               0 1970-01-01 00:00 a9543c9dd4b34977774394feb7b1aae00bc39311d31cfe02c5c8165abef0c266/layer.tar -> ../02a390e9ccf0f6ab23ff00a9f8957f983f5e9892fb5f55f120e1bcdb68ec3cdf.tar
-r--r--r-- 0/0               3 1970-01-01 00:00 a9543c9dd4b34977774394feb7b1aae00bc39311d31cfe02c5c8165abef0c266/VERSION
-r--r--r-- 0/0             452 1970-01-01 00:00 a9543c9dd4b34977774394feb7b1aae00bc39311d31cfe02c5c8165abef0c266/json
-r--r--r-- 0/0             103 1970-01-01 00:00 repositories


[root@servera ~]# podman rmi alpine/htop
Untagged: localhost/alpine/htop:latest
Deleted: afd869d36178fdb6a5d4ac9958ccc932938d2f3d30ab3165c258bb75d8a1ee97


[root@servera ~]# podman rmi docker.io/alpine
Untagged: docker.io/library/alpine:latest
Deleted: d6e46aa2470df1d32034c6707c8041158b652f38d2a9ae3d7ad7e7532d22ebe0


[root@servera ~]# podman images
REPOSITORY   TAG   IMAGE ID   CREATED   SIZE


[root@servera ~]# podman load -i alpine_htop.tar
Getting image source signatures
Copying blob 02a390e9ccf0 done  
Copying blob ace0eda3e3be done  
Copying config afd869d361 done  
Writing manifest to image destination
Storing signatures
Loaded image(s): localhost/alpine/htop:latest


[root@servera ~]# podman images
REPOSITORY              TAG      IMAGE ID       CREATED         SIZE
localhost/alpine/htop   latest   afd869d36178   8 minutes ago   8.47 MB

```

_**Congratulations! this concludes the exercise.**_
