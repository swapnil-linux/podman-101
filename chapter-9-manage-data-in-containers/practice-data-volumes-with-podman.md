# Practice: Data volumes with Podman

1. pull mysql image from registry.access.redhat.com and inspect it

```
podman pull registry.access.redhat.com/rhscl/mysql-57-rhel7
```

```
[root@servera ~]# podman inspect registry.access.redhat.com/rhscl/mysql-57-rhel7|grep User
            "User": "27",
```

this shows that the application will run as userid 27, and the volume needs to be writable by UID 27

2\. create a folder on host and make UID 27 as the owner. Also check the existing SELinux label

```
[root@servera ~]# mkdir /opt/mysqldata
[root@servera ~]# chown 27 /opt/mysqldata

[root@servera ~]# ls -lZd /opt/mysqldata
drwxr-xr-x. 27 root unconfined_u:object_r:usr_t:s0   /opt/mysqldata

```

the existing label is usr\_t _and needs to be changed to_ container\_file\_t,&#x20;

3\. create the mysql container with volume mount using z flag to fix SELinux label and proper environment variables.

```
podman run -d --name=mysql -v /opt/mysqldata:/var/lib/mysql/data:z -e MYSQL_USER=user1 -e MYSQL_PASSWORD=mypass1234 -e MYSQL_DATABASE=friends registry.access.redhat.com/rhscl/mysql-57-rhel7  
```

4\. inspect the label of the host folder and files created within it.

```
[root@servera ~]# ls -ldZ /opt/mysqldata
drwxr-xr-x. 27 root system_u:object_r:container_file_t:s0 /opt/mysqldata
```

```
[root@servera ~]# ls -lZ /opt/mysqldata/
-rw-r-----. 27 27 system_u:object_r:container_file_t:s0 744767d91c14.pid
-rw-r-----. 27 27 system_u:object_r:container_file_t:s0 auto.cnf
..
..
..
..
-rw-r--r--. 27 27 system_u:object_r:container_file_t:s0 server-cert.pem
-rw-------. 27 27 system_u:object_r:container_file_t:s0 server-key.pem
drwxr-x---. 27 27 system_u:object_r:container_file_t:s0 sys
```

5\. Create a table in insert few records

```
[root@servera ~]# podman exec -it mysql bash


bash-4.2$ mysql -uuser1 -pmypass1234 friends
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

6\. exit the container and kill and delete it

```
mysql> exit
Bye
bash-4.2$ exit
exit

[root@servera ~]# podman kill mysql
744767d91c14ace91943f0a5ece303612b558e79674a6829dd1c51eb54ecfa42


[root@servera ~]# podman rm mysql
744767d91c14ace91943f0a5ece303612b558e79674a6829dd1c51eb54ecfa42
[root@servera ~]# 

```

7\. create another container mounting the same host folder as volume, and you should be able to see the table and record exists

```
podman run -d --name=mysql-new -v /opt/mysqldata:/var/lib/mysql/data:z -e MYSQL_USER=user1 -e MYSQL_PASSWORD=mypass1234 -e MYSQL_DATABASE=friends registry.access.redhat.com/rhscl/mysql-57-rhel7  
```

```
[root@servera ~]# podman exec -it mysql-new bash

bash-4.2$ mysql -uuser1 -pmypass1234 friends

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
[root@servera ~]# podman kill mysql-new 
5b4af7a53e635ce22e3c6408059c4c275d9d44f5457ef6e3acc8e6160c3a5bbb

[root@servera ~]# podman system prune --all -f --volumes 
Deleted Pods
Deleted Containers
5b4af7a53e635ce22e3c6408059c4c275d9d44f5457ef6e3acc8e6160c3a5bbb
Deleted Volumes
Deleted Images
60726b33a00a2c3be60e25c3270a34a9b147db86602f05a71988a1c92a70cebc
[root@servera ~]# 
```

_**Congratulations! this completes the exercise.**_
