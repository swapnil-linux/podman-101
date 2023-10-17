# Practice: Creating a Wordpress Pod

1. create a pod mapping port 38080 to port 80. Later if you then spin up a container listening on port 80, you'd have connectivity.

```
[root@servera ~]# podman pod create --name my-pod -p 38080:80
c8f626efa47ab606b1152b4c1b7e4956fa884a7a519c2b92def9698a8a9f1f96


[root@servera ~]# podman pod ps
POD ID         NAME     STATUS    CREATED          # OF CONTAINERS   INFRA ID
c8f626efa47a   my-pod   Created   11 seconds ago   1                 c11f276a3505

```

2\. use `podman run`, to create a database and wordpress in the same pod container but don't map a port. This makes more sense when you have more than one container to work with, so we are going to create a database container and then a WordPress container.

```
podman run -d --restart=always --pod=my-pod -e MYSQL_ROOT_PASSWORD="myrootpass" -e MYSQL_DATABASE="wp" -e MYSQL_USER="wordpress" -e MYSQL_PASSWORD="w0rdpr3ss" --name=wptest-db docker.io/mariadb
```

```
podman run -d --restart=always --pod=my-pod -e WORDPRESS_DB_NAME="wp" -e WORDPRESS_DB_USER="wordpress" -e WORDPRESS_DB_PASSWORD="w0rdpr3ss" -e WORDPRESS_DB_HOST="127.0.0.1" --name wptest-web docker.io/wordpress
```

{% hint style="info" %}
Notice that we pointed the `wordpress_db_host` in the `env:` to localhost. That's because the WordPress container is going to find the database container on the local host.
{% endhint %}



```
[root@servera ~]# podman pod ps
POD ID         NAME     STATUS    CREATED          # OF CONTAINERS   INFRA ID
c8f626efa47a   my-pod   Running   10 minutes ago   3                 c11f276a3505


[root@servera ~]# podman ps -a --pod
CONTAINER ID  IMAGE                               COMMAND               CREATED         STATUS             PORTS                  NAMES               POD
caa95117fd9e  docker.io/library/wordpress:latest  apache2-foregroun...  22 seconds ago  Up 22 seconds ago  0.0.0.0:38080->80/tcp  wptest-web          c8f626efa47a
be5f75f56c43  docker.io/library/mariadb:latest    mysqld                4 minutes ago   Up 4 minutes ago   0.0.0.0:38080->80/tcp  wptest-db           c8f626efa47a
c11f276a3505  k8s.gcr.io/pause:3.1                                      10 minutes ago  Up 4 minutes ago   0.0.0.0:38080->80/tcp  c8f626efa47a-infra  c8f626efa47a
[root@servera ~]# 

```

3\. In your browser, you can get to the WordPress setup page in http://yoursystem:38080

{% hint style="info" %}
leave the pods and containers running we will need it in the next exercise
{% endhint %}





