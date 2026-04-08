# Practice: Creating a multi-container application

1. use podman pull to pull mysql image

```
$ podman pull docker.io/mysql:8.0
Trying to pull docker.io/library/mysql:8.0...
Getting image source signatures
Copying blob a43f41a44c48 done   |
Copying blob 938c64119969 done   |
Copying blob 852e50cd189d done   |
Copying blob b79b040de953 done   |
Copying config ae0658fdba done   |
Writing manifest to image destination
ae0658fdbad5fb1c9413c998d8a573eeb5d16713463992005029c591e6400d02

```

2\. create a mysql container using few environment variables

```
podman run --name=friends-db -p 3306:3306 -e MYSQL_USER=user1 -e MYSQL_PASSWORD=pass1234 -e MYSQL_DATABASE=friends -e MYSQL_RANDOM_ROOT_PASSWORD=yes -d docker.io/mysql:8.0
```

```
$ podman ps
CONTAINER ID  IMAGE                        COMMAND  CREATED        STATUS            PORTS                   NAMES
335d3413b189  docker.io/library/mysql:8.0  mysqld   4 seconds ago  Up 4 seconds ago  0.0.0.0:3306->3306/tcp  friends-db

```

```
$ podman logs friends-db
...
[Server] /usr/sbin/mysqld: ready for connections.
Version: '8.0.40'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server - GPL

```

3\. Find the IP Address of mysql container, we will need this in next step

```
$ podman inspect friends-db |grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "10.88.0.21",

```

4\. create a php container with similar environment variables as mysql containers, and also create a hosts entry for mysql using `--add-host` option

```
podman run --name=friends-app -p 38080:80 -e MYSQL_USER=user1 -e MYSQL_PASSWORD=pass1234 -e MYSQL_DATABASE=friends --add-host=mysql:10.88.0.21 -d docker.io/swapnillinux/apache-php
```

```
$ podman ps
CONTAINER ID  IMAGE                                     COMMAND      CREATED        STATUS            PORTS                   NAMES
c9da8a8f82ce  docker.io/swapnillinux/apache-php:latest  /startup.sh  3 seconds ago  Up 3 seconds ago  0.0.0.0:38080->80/tcp   friends-app
335d3413b189  docker.io/library/mysql:8.0               mysqld       2 minutes ago  Up 2 minutes ago  0.0.0.0:3306->3306/tcp  friends-db

```

5\. Clone the php-app app using git and copy the index.php in the php container

{% hint style="info" %}
if git is not installed, install using&#x20;

`dnf install git -y`

or&#x20;

`apt install git -y`
{% endhint %}

```
$ git clone https://github.com/swapnil-linux/php-app
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
$ podman exec -it friends-db bash

bash-4.4# mysql -u user1 -ppass1234 friends
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 7
Server version: 8.0.40 MySQL Community Server - GPL

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
$ podman kill friends-db friends-app 
c9da8a8f82ce
335d3413b189

$ podman rm friends-db friends-app 
335d3413b189
c9da8a8f82ce

$ podman rmi docker.io/mysql:8.0 docker.io/swapnillinux/apache-php
Untagged: docker.io/library/mysql:8.0
Deleted: ae0658fdbad5fb1c9413c998d8a573eeb5d16713463992005029c591e6400d02
Untagged: docker.io/swapnillinux/apache-php:latest
Deleted: 26d33e5b4a9188118a1cdea4cfd4dba73453a20304b099e8dc4fe548aa0ef3a7
```

{% hint style="info" %}
view the index.php to understand how the php application is able to connect to the database
{% endhint %}
