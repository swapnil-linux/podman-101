# Practice: Data volumes with Podman

1. Pull mysql image and inspect it

```
podman pull docker.io/mysql:8.0
```

```
$ podman inspect docker.io/mysql:8.0 |grep User
            "User": "mysql",
```

This shows that the application will run as the mysql user, and the volume needs to be writable by that user.

2\. Create a folder on host and set proper ownership. Also check the existing SELinux label

```
$ sudo mkdir /opt/mysqldata
$ sudo chown 999 /opt/mysqldata

$ ls -lZd /opt/mysqldata
drwxr-xr-x. 999 root unconfined_u:object_r:usr_t:s0   /opt/mysqldata

```

The existing label is usr\_t _and needs to be changed to_ container\_file\_t.&#x20;

3\. Create the mysql container with volume mount using z flag to fix SELinux label and proper environment variables.

```
sudo podman run -d --name=mysql -v /opt/mysqldata:/var/lib/mysql:z -e MYSQL_USER=user1 -e MYSQL_PASSWORD=mypass1234 -e MYSQL_DATABASE=friends -e MYSQL_RANDOM_ROOT_PASSWORD=yes docker.io/mysql:8.0
```

4\. Inspect the label of the host folder and files created within it.

```
$ ls -ldZ /opt/mysqldata
drwxr-xr-x. 999 root system_u:object_r:container_file_t:s0 /opt/mysqldata
```

```
$ ls -lZ /opt/mysqldata/
-rw-r-----. 999 999 system_u:object_r:container_file_t:s0 auto.cnf
..
..
-rw-r--r--. 999 999 system_u:object_r:container_file_t:s0 server-cert.pem
-rw-------. 999 999 system_u:object_r:container_file_t:s0 server-key.pem
drwxr-x---. 999 999 system_u:object_r:container_file_t:s0 sys
```

5\. Create a table and insert a few records

```
$ sudo podman exec -it mysql bash

bash-4.4# mysql -uuser1 -pmypass1234 friends
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

6\. Exit the container and kill and delete it

```
mysql> exit
Bye
bash-4.4# exit
exit

$ sudo podman kill mysql
mysql

$ sudo podman rm mysql
mysql

```

7\. Create another container mounting the same host folder as volume, and you should be able to see the table and records still exist

```
sudo podman run -d --name=mysql-new -v /opt/mysqldata:/var/lib/mysql:z -e MYSQL_USER=user1 -e MYSQL_PASSWORD=mypass1234 -e MYSQL_DATABASE=friends -e MYSQL_RANDOM_ROOT_PASSWORD=yes docker.io/mysql:8.0
```

```
$ sudo podman exec -it mysql-new bash

bash-4.4# mysql -uuser1 -pmypass1234 friends

mysql> select * from MyGuests;
+----+-----------+---------------+
| id | firstname | lastname      |
+----+-----------+---------------+
|  1 | Chandler  | Bing          |
|  2 | Rachel    | Green         |
|  3 | Monica    | Geller        |
|  4 | Dr. Ross  | Geller        |
|  5 | Joey      | Tribbiani Jr. |
|  6 | Phoebe    | Buffay        |
+----+-----------+---------------+
6 rows in set (0.00 sec)

mysql> 

```

8\. Final clean up

```
$ sudo podman kill mysql-new 
mysql-new

$ sudo podman system prune --all -f --volumes 
```

_**Congratulations! this completes the exercise.**_
