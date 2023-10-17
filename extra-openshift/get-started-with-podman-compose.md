# Get started with Podman-Compose

podman-compose is an implementation of `docker-compose` with [Podman](https://podman.io/) backend. The main objective of this project is to be able to run `docker-compose.yml` unmodified and rootless. This project is aimed to provide drop-in replacement for `docker-compose`, and it's very useful for certain cases because:

* can run rootless
* only depend on `podman` and Python3 and [PyYAML](https://pyyaml.org/)
* no daemon, no setup.
* can be used by developers to run single-machine containerized stacks using single familiar YAML file

1. Install python3 and dependencies

```
yum install -y python3 python36-PyYAML PyYAML
```

2\. download podman-compose

```

curl -o /usr/bin/podman-compose https://raw.githubusercontent.com/containers/podman-compose/devel/podman_compose.py
chmod +x /usr/bin/podman-compose
```

3\. create a working directory

```
mkdir awx
cd awx
```

4\. download a docker-compose file to deploy AWX

```
wget -O docker-compose.yml https://raw.githubusercontent.com/containers/podman-compose/devel/examples/awx3/docker-compose.yml
```

5\. review the docker-compose.yml file and bring up the services

```
podman-compose up -d
```

```
[root@servera awx]# podman-compose ps
using podman version: podman version 1.6.4
podman ps -a --filter label=io.podman.compose.project=awx
CONTAINER ID  IMAGE                               COMMAND               CREATED             STATUS                 PORTS                    NAMES
be234c4c2305  docker.io/ansible/awx_task:3.0.1    /bin/sh -c /usr/b...  11 seconds ago      Up 11 seconds ago      0.0.0.0:38080->8052/tcp  awx_awx_task_1
1d92033d9e54  docker.io/ansible/awx_web:3.0.1     /bin/sh -c /usr/b...  23 seconds ago      Up 22 seconds ago      0.0.0.0:38080->8052/tcp  awx_awx_web_1
3fb2d19ec984  docker.io/library/memcached:alpine  memcached             About a minute ago  Up About a minute ago  0.0.0.0:38080->8052/tcp  awx_memcached_1
65111edef5e9  docker.io/library/rabbitmq:3        rabbitmq-server       About a minute ago  Up About a minute ago  0.0.0.0:38080->8052/tcp  awx_rabbitmq_1
78ba4c7e8fe9  docker.io/library/postgres:9.6      postgres              About a minute ago  Up About a minute ago  0.0.0.0:38080->8052/tcp  awx_postgres_1
0
[root@servera awx]# 
```

6\. try accessing AWX using the exposed port. User `admin` as user and `password` as the password

![](<../.gitbook/assets/Screen Shot 2020-12-16 at 12.39.18 pm.png>)

7\. final clean up

```
[root@servera awx]# podman-compose down

using podman version: podman version 1.6.4
podman stop -t 10 awx_postgres_1
78ba4c7e8fe9bb3b89e9893b8d72903960014724062665baddcf187d8365f830
0
podman stop -t 10 awx_rabbitmq_1
65111edef5e9c25c89952ca51e15b675ffe5ca09ca8282324ac7b911b39a5254
0
podman stop -t 10 awx_memcached_1
3fb2d19ec98421717063025ba5c32dbdb25057b8a7375e8185859b413deed717
0
podman stop -t 10 awx_awx_web_1
1d92033d9e54076340845c0df7202b0367fb4b72624089c9ebd07c7cdacf75e7
0
podman stop -t 10 awx_awx_task_1
be234c4c230508e4b72e6b6d29553fbcef37581662c20b840c7cad7c132513fa
0
podman rm awx_postgres_1
78ba4c7e8fe9bb3b89e9893b8d72903960014724062665baddcf187d8365f830
0
podman rm awx_rabbitmq_1
65111edef5e9c25c89952ca51e15b675ffe5ca09ca8282324ac7b911b39a5254
0
podman rm awx_memcached_1
3fb2d19ec98421717063025ba5c32dbdb25057b8a7375e8185859b413deed717
0
podman rm awx_awx_web_1
1d92033d9e54076340845c0df7202b0367fb4b72624089c9ebd07c7cdacf75e7
0
podman rm awx_awx_task_1
be234c4c230508e4b72e6b6d29553fbcef37581662c20b840c7cad7c132513fa
0
podman pod rm awx
6739a9b472dbb9adb8fa92fed63f246befed61056a6888bd398723c00e61880c
0
[root@servera awx]# 

```
