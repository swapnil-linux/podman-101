# Practice: Creating a multi-container application

1. use podman pull to pull mysql image

```
[root@servera ~]# podman pull docker.io/mysql:5.7
Trying to pull docker.io/mysql:5.7...
Getting image source signatures
Copying blob a43f41a44c48 done  
Copying blob 938c64119969 done  
Copying blob 852e50cd189d done  
Copying blob b79b040de953 done  
Copying blob 29969ddb0ffb done  
Copying blob 5cdd802543a3 done  
Copying blob 36bd6224d58f done  
Copying blob 7689ec51a0d9 done  
Copying blob cab9d3fa4c8c done  
Copying blob 1b741e1c47de done  
Copying blob aac9d11987ac done  
Copying config ae0658fdba done  
Writing manifest to image destination
Storing signatures
ae0658fdbad5fb1c9413c998d8a573eeb5d16713463992005029c591e6400d02

```

2\. create a mysql container using few environment variables

```
podman run --name=friends-db -p 3306:3306 -e MYSQL_USER=user1 -e MYSQL_PASSWORD=pass1234 -e MYSQL_DATABASE=friends -e MYSQL_RANDOM_ROOT_PASSWORD=yes -d docker.io/mysql:5.7
```

```
[root@servera ~]# podman ps
CONTAINER ID  IMAGE                        COMMAND  CREATED        STATUS            PORTS                   NAMES
335d3413b189  docker.io/library/mysql:5.7  mysqld   4 seconds ago  Up 4 seconds ago  0.0.0.0:3306->3306/tcp  friends-db
[root@servera ~]#

```

```
[root@servera ~]# podman logs friends-db
2020-12-09 10:00:30+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 5.7.32-1debian10 started.
2020-12-09 10:00:30+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2020-12-09 10:00:30+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 5.7.32-1debian10 started.
2020-12-09 10:00:30+00:00 [Note] [Entrypoint]: Initializing database files
...
...
...
2020-12-09T10:00:40.496529Z 0 [Note] Event Scheduler: Loaded 0 events
2020-12-09T10:00:40.496783Z 0 [Note] mysqld: ready for connections.
Version: '5.7.32'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)

```

3\. Find the IP Address of mysql container, we will need this in next step

```
[root@servera ~]# podman inspect friends-db |grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "10.88.0.21",

```

4\. create a php container with similer environment variables as mysql containers, and also create a hosts entry for mysql using `--add-host` option

```
podman run --name=friends-app -p 38080:80 -e MYSQL_USER=user1 -e MYSQL_PASSWORD=pass1234 -e MYSQL_DATABASE=friends --add-host=mysql:10.88.0.21 -d docker.io/swapnillinux/apache-php
```

```
[root@servera ~]# podman ps
CONTAINER ID  IMAGE                                     COMMAND      CREATED        STATUS            PORTS                   NAMES
c9da8a8f82ce  docker.io/swapnillinux/apache-php:latest  /startup.sh  3 seconds ago  Up 3 seconds ago  0.0.0.0:38080->80/tcp   friends-app
335d3413b189  docker.io/library/mysql:5.7               mysqld       2 minutes ago  Up 2 minutes ago  0.0.0.0:3306->3306/tcp  friends-db
[root@servera ~]#

```

5\. Clone the php-app app using git and copy the index.php in the php container

{% hint style="info" %}
if git is not installed, install using&#x20;

`yum install git -y`

or&#x20;

`apt install git -y`
{% endhint %}

```
[root@servera ~]# git clone https://github.com/swapnil-linux/php-app
Cloning into 'php-app'...
remote: Enumerating objects: 5, done.
remote: Counting objects: 100% (5/5), done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 50 (delta 1), reused 3 (delta 1), pack-reused 45
Unpacking objects: 100% (50/50), done.
```

```
podman cp php-app/index.php friends-app:/var/www/html/index.php
```

6\. access the application using browser and pointing to http://yoursystemip:38080

7\. create a table in mysql and insert few records.

```
[root@servera ~]# podman exec -it friends-db bash

root@335d3413b189:/# mysql -u user1 -ppass1234 friends
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 7
Server version: 5.7.32 MySQL Community Server (GPL)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

```
CREATE TABLE IF NOT EXISTS MyGuests (id INT AUTO_INCREMENT PRIMARY KEY, firstname VARCHAR(255) NOT NULL, lastname VARCHAR(255) NOT NULL )  ENGINE=INNODB;insert into MyGuests (firstname,lastname) VALUES ("Chandler","Bing");
insert into MyGuests (firstname,lastname) VALUES ("Rachel","Green");
insert into MyGuests (firstname,lastname) VALUES ("Monica","Geller");
insert into MyGuests (firstname,lastname) VALUES ("Dr. Ross","Geller");
insert into MyGuests (firstname,lastname) VALUES ("Joey","Tribbiani Jr.");
insert into MyGuests (firstname,lastname) VALUES ("Phoebe","Buffay");
```

8\. refresh the page on browser to see the changes.

9\. final cleanup

```
[root@servera ~]# podman kill friends-db friends-app 
11f2fd489bb90ed60888693998d25b57b101d7ccb072583f1071d3ededf05e63
335d3413b18911e5a1f717065be6d2055fab95d2195aecdcb01bee43af8e824e


[root@servera ~]# podman rm friends-db friends-app 
335d3413b18911e5a1f717065be6d2055fab95d2195aecdcb01bee43af8e824e
11f2fd489bb90ed60888693998d25b57b101d7ccb072583f1071d3ededf05e63


[root@servera ~]# podman rmi docker.io/mysql:5.7 docker.io/swapnillinux/apache-php
Untagged: docker.io/library/mysql:5.7
Deleted: ae0658fdbad5fb1c9413c998d8a573eeb5d16713463992005029c591e6400d02
Untagged: docker.io/swapnillinux/apache-php:latest
Deleted: 26d33e5b4a9188118a1cdea4cfd4dba73453a20304b099e8dc4fe548aa0ef3a7
[root@servera ~]# 
```

{% hint style="info" %}
view the index.php to understand how the php application is able to connect to the database
{% endhint %}
