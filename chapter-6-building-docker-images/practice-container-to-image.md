# Practice: Container to Image

1: Create an alpine container and install htop

```
$ podman run -it --name=testcommit docker.io/alpine sh
Trying to pull docker.io/library/alpine:latest...
Getting image source signatures
Copying blob c6a83fedfae6 done   |
Copying config 1d34ffeaf1 done   |
Writing manifest to image destination


/ # htop
sh: htop: not found


/ # apk update
fetch https://dl-cdn.alpinelinux.org/alpine/v3.21/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.21/community/x86_64/APKINDEX.tar.gz
OK: 24942 distinct packages available


/ # apk add htop
(1/3) Installing ncurses-terminfo-base (6.5_p20241006-r3)
(2/3) Installing ncurses-libs (6.5_p20241006-r3)
(3/3) Installing htop (3.3.0-r0)
OK: 8 MiB in 18 packages
/ # 


```

2\. Test running htop, q to exit htop and exit the container.

```
/ # htop
/ # exit
```

3\. Inspect the changes done in the container layer using podman diff command

```
$ podman diff testcommit
C /etc
C /etc/apk
C /etc/apk/world
A /etc/terminfo
A /etc/terminfo/a
..
..
.. 
```

4\. Create a new image named alpine/htop from a container's changes using podman commit command. Remove the testcommit container after saving it as image.

```
$ podman commit testcommit alpine/htop
Getting image source signatures
Copying blob ace0eda3e3be skipped: already exists  
Copying blob 02a390e9ccf0 done   |
Copying config afd869d361 done   |
Writing manifest to image destination
Storing signatures
afd869d36178fdb6a5d4ac9958ccc932938d2f3d30ab3165c258bb75d8a1ee97

$ podman images
REPOSITORY                 TAG      IMAGE ID       CREATED              SIZE
localhost/alpine/htop      latest   afd869d36178   About a minute ago   10.5 MB
docker.io/library/alpine   latest   1d34ffeaf190   4 weeks ago          7.79 MB

$ podman rm testcommit 
testcommit
```

5\. Test the new image by creating a temporary container, without creating the PID namespace and starting htop process as a UID 1234

```
podman run --rm -it -u 1234 --pid=host alpine/htop htop
```

6\. Save an image as tar file using podman save command. Delete the image and load it back using podman load command.

```
$ podman save alpine/htop > alpine_htop.tar

$ tar tvf alpine_htop.tar|head
-r--r--r-- 0/0         5843456 1970-01-01 00:00 ace0eda3e3be35a979cec764a3321b4c7d0b9e4bb3094d20d3ff6782961a8d54.tar
-r--r--r-- 0/0         2625024 1970-01-01 00:00 02a390e9ccf0f6ab23ff00a9f8957f983f5e9892fb5f55f120e1bcdb68ec3cdf.tar
-r--r--r-- 0/0             782 1970-01-01 00:00 afd869d36178fdb6a5d4ac9958ccc932938d2f3d30ab3165c258bb75d8a1ee97.json


$ podman rmi alpine/htop
Untagged: localhost/alpine/htop:latest
Deleted: afd869d36178fdb6a5d4ac9958ccc932938d2f3d30ab3165c258bb75d8a1ee97


$ podman rmi docker.io/alpine
Untagged: docker.io/library/alpine:latest
Deleted: 1d34ffeaf190be23d3de5a8de0a85e3b15a3da1b9b7b21ab5fe8717ce0e3b12e


$ podman images
REPOSITORY   TAG   IMAGE ID   CREATED   SIZE


$ podman load -i alpine_htop.tar
Getting image source signatures
Copying blob 02a390e9ccf0 done   |
Copying blob ace0eda3e3be done   |
Copying config afd869d361 done   |
Writing manifest to image destination
Storing signatures
Loaded image(s): localhost/alpine/htop:latest


$ podman images
REPOSITORY              TAG      IMAGE ID       CREATED         SIZE
localhost/alpine/htop   latest   afd869d36178   8 minutes ago   10.5 MB

```

_**Congratulations! this concludes the exercise.**_
