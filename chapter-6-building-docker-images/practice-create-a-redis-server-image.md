# Practice: Create a redis server image

1: Create a working directory as alpine-redis

```
[root@servera ~]# mkdir alpine-redis
[root@servera ~]# cd alpine-redis
[root@servera alpine-redis]#
```

2\. Create a file named startup.sh with the below content in alpine-redis and make it executable&#x20;

```
#!/bin/bash
set -e

if [ "$1" = 'redis-server' ]; then
  chown -R redis .
  exec gosu redis "$@"
fi

exec "$@"
```

```
[root@servera alpine-redis]# chmod +x startup.sh 

[root@servera alpine-redis]# ls -l
total 4
-rwxr-xr-x. 1 root root 111 Dec 10 23:37 startup.sh
[root@servera alpine-redis]# 
```

3\. Create Dockerfile with the below content.

```
FROM docker.io/alpine:latest

RUN apk --update add bash nano curl redis && rm -rf /var/cache/apk/*

ADD "https://github.com/tianon/gosu/releases/download/1.9/gosu-amd64" /usr/local/bin/gosu

RUN chmod 755 /usr/local/bin/gosu && mkdir /data && chown redis:redis /data

COPY startup.sh /

ENTRYPOINT ["/startup.sh"]

WORKDIR /data

EXPOSE 6379

VOLUME /data

CMD [ "redis-server" ]

```

4\. Build the image using podman build command

```
[root@servera alpine-redis]# podman build -t alpine/redis .

STEP 1: FROM docker.io/alpine:latest
Getting image source signatures
Copying blob 188c0c94c7c5 done  
Copying config d6e46aa247 done  
Writing manifest to image destination
Storing signatures
STEP 2: RUN apk --update add bash nano curl redis && rm -rf /var/cache/apk/*
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/community/x86_64/APKINDEX.tar.gz
(1/11) Installing ncurses-terminfo-base (6.2_p20200523-r0)
(2/11) Installing ncurses-libs (6.2_p20200523-r0)
(3/11) Installing readline (8.0.4-r0)
(4/11) Installing bash (5.0.17-r0)
Executing bash-5.0.17-r0.post-install
(5/11) Installing ca-certificates (20191127-r4)
(6/11) Installing nghttp2-libs (1.41.0-r0)
(7/11) Installing libcurl (7.69.1-r2)
(8/11) Installing curl (7.69.1-r2)
(9/11) Installing libmagic (5.38-r0)
(10/11) Installing nano (4.9.3-r0)
(11/11) Installing redis (5.0.9-r0)
Executing redis-5.0.9-r0.pre-install
Executing redis-5.0.9-r0.post-install
Executing busybox-1.31.1-r19.trigger
Executing ca-certificates-20191127-r4.trigger
OK: 17 MiB in 25 packages
5de539ec46faca67bcda001e97edef12c421e2b37b8d523f2f8fe815e6753b0d
STEP 3: ADD "https://github.com/tianon/gosu/releases/download/1.9/gosu-amd64" /usr/local/bin/gosu
f019249ac3b262677144c44da58b821e57f86e23166c3f74c1b538a662e28b04
STEP 4: RUN chmod 755 /usr/local/bin/gosu && mkdir /data && chown redis:redis /data
1f3ce3d4209b827409b461b5de999cbc8db56785aed84a15cf7d3aa5a4910015
STEP 5: COPY startup.sh /
5340343a6fc98c5557decdf06d42d39951bf4362b72f7e737f229a5c5293af9a
STEP 6: ENTRYPOINT ["/startup.sh"]
401b23e9f9ab0cd573fe28df498400bf03604b678eb6158d44bc13c345ce5f66
STEP 7: WORKDIR /data
7386985a322855f1d1ae2e7f69bc24b8c5f30440485c563e3bc17ac99e5e25ea
STEP 8: EXPOSE 6379
334768fff228acfc99502564768c5455ef716dced9db4076944fda57b7bf911c
STEP 9: VOLUME /data
618e8f8fae1a87159c06747dc0b799decb9c45d81065b89391c510c34356a271
STEP 10: CMD [ "redis-server" ]
STEP 11: COMMIT alpine/redis
6e859d2b97b28130e0bb945ee595057f305eea1b236d34e6f5dd1f23e7d87245
6e859d2b97b28130e0bb945ee595057f305eea1b236d34e6f5dd1f23e7d87245

```

```
[root@servera alpine-redis]# podman images alpine/redis
REPOSITORY               TAG      IMAGE ID       CREATED         SIZE
localhost/alpine/redis   latest   6e859d2b97b2   2 minutes ago   20.7 MB
[root@servera alpine-redis]# 
```

5\. Create a redis container using the newly built image

```
[root@servera ~]# podman run --name=myredis -p 6379:6379 -d alpine/redis
8dc667e97626a6959e9cc2bbd482c77fc9dfe9ff0408a617d6f6d74288c7e466


[root@servera ~]# podman ps
CONTAINER ID  IMAGE                          COMMAND       CREATED        STATUS            PORTS                   NAMES
8dc667e97626  localhost/alpine/redis:latest  redis-server  2 seconds ago  Up 2 seconds ago  0.0.0.0:6379->6379/tcp  myredis


[root@servera ~]# podman logs myredis
1:C 10 Dec 2020 23:47:38.620 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 10 Dec 2020 23:47:38.620 # Redis version=5.0.9, bits=64, commit=869dcbdc, modified=0, pid=1, just started
1:C 10 Dec 2020 23:47:38.620 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
1:M 10 Dec 2020 23:47:38.622 * Running mode=standalone, port=6379.
1:M 10 Dec 2020 23:47:38.623 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
1:M 10 Dec 2020 23:47:38.623 # Server initialized
1:M 10 Dec 2020 23:47:38.623 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
1:M 10 Dec 2020 23:47:38.623 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
1:M 10 Dec 2020 23:47:38.623 * Ready to accept connections

```

6\. Create another container using the same image, connecting to the network namespace of the first container. And, run a benchmark test.

```
podman run -it --rm --net=container:myredis alpine/redis redis-benchmark -h 127.0.0.1 -p 6379 -n 100
```

_**Congratulations! this concludes the exercise.**_
