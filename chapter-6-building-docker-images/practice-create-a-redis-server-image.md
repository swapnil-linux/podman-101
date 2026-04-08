# Practice: Create a redis server image

1: Create a working directory as alpine-redis

```
$ mkdir alpine-redis
$ cd alpine-redis
```

2\. Create a file named startup.sh with the below content in alpine-redis and make it executable&#x20;

```
#!/bin/sh
set -e

if [ "$1" = 'redis-server' ]; then
  chown -R redis .
  exec su-exec redis "$@"
fi

exec "$@"
```

```
$ chmod +x startup.sh 

$ ls -l
total 4
-rwxr-xr-x. 1 root root 117 startup.sh
```

3\. Create a Containerfile with the below content.

```
FROM docker.io/alpine:latest

RUN apk --update add bash curl redis su-exec && rm -rf /var/cache/apk/*

RUN mkdir /data && chown redis:redis /data

COPY startup.sh /

ENTRYPOINT ["/startup.sh"]

WORKDIR /data

EXPOSE 6379

VOLUME /data

CMD [ "redis-server" ]

```

4\. Build the image using podman build command

```
$ podman build -t alpine/redis .

STEP 1/9: FROM docker.io/alpine:latest
Trying to pull docker.io/library/alpine:latest...
Getting image source signatures
Copying blob c6a83fedfae6 done   |
Copying config 1d34ffeaf1 done   |
Writing manifest to image destination
STEP 2/9: RUN apk --update add bash curl redis su-exec && rm -rf /var/cache/apk/*
fetch https://dl-cdn.alpinelinux.org/alpine/v3.21/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.21/community/x86_64/APKINDEX.tar.gz
(1/11) Installing ncurses-terminfo-base (6.5_p20241006-r3)
...
(11/11) Installing redis (7.2.7-r0)
OK: 17 MiB in 25 packages
STEP 3/9: RUN mkdir /data && chown redis:redis /data
STEP 4/9: COPY startup.sh /
STEP 5/9: ENTRYPOINT ["/startup.sh"]
STEP 6/9: WORKDIR /data
STEP 7/9: EXPOSE 6379
STEP 8/9: VOLUME /data
STEP 9/9: CMD [ "redis-server" ]
COMMIT alpine/redis
--> 6e859d2b97b2
6e859d2b97b28130e0bb945ee595057f305eea1b236d34e6f5dd1f23e7d87245

```

```
$ podman images alpine/redis
REPOSITORY               TAG      IMAGE ID       CREATED         SIZE
localhost/alpine/redis   latest   6e859d2b97b2   2 minutes ago   18.4 MB
```

5\. Create a redis container using the newly built image

```
$ podman run --name=myredis -p 6379:6379 -d alpine/redis
8dc667e97626a6959e9cc2bbd482c77fc9dfe9ff0408a617d6f6d74288c7e466


$ podman ps
CONTAINER ID  IMAGE                          COMMAND       CREATED        STATUS            PORTS                   NAMES
8dc667e97626  localhost/alpine/redis:latest  redis-server  2 seconds ago  Up 2 seconds ago  0.0.0.0:6379->6379/tcp  myredis


$ podman logs myredis
...
* Running mode=standalone, port=6379.
* Server initialized
* Ready to accept connections tcp

```

6\. Create another container using the same image, connecting to the network namespace of the first container. And, run a benchmark test.

```
podman run -it --rm --net=container:myredis alpine/redis redis-benchmark -h 127.0.0.1 -p 6379 -n 100
```

_**Congratulations! this concludes the exercise.**_
