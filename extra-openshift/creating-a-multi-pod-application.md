# Deploying a multi-pod application on OpenShift

1. make sure you are logged in by running `oc whoami` command

```
[student@servera ~]$ oc whoami -c
user15-testproject/api-swapnil-mask365-com:6443/user15
```

2\. create a mysql pod with proper environment variables

```
[student@servera ~]$ oc new-app --name=mysql --docker-image=registry.access.redhat.com/rhscl/mysql-57-rhel7 -e MYSQL_USER=user1 -e MYSQL_PASSWORD=redhat -e MYSQL_DATABASE=friends


--> Found container image 60726b3 (14 months old) from registry.access.redhat.com for "registry.access.redhat.com/rhscl/mysql-57-rhel7"

    MySQL 5.7 
    --------- 
    MySQL is a multi-user, multi-threaded SQL database server. The container image provides a containerized packaging of the MySQL mysqld daemon and client application. The mysqld server daemon accepts connections from clients and provides access to content from MySQL databases on behalf of the clients.

    Tags: database, mysql, mysql57, rh-mysql57

    * An image stream tag will be created as "mysql:latest" that will track this image

--> Creating resources ...
    imagestream.image.openshift.io "mysql" created
    deployment.apps "mysql" created
    service "mysql" created
--> Success
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose service/mysql' 
    Run 'oc status' to view your app.
```

3\. create application container  with the same environment variables and source code from https://github.com/swapnil-linux/php-app

```
[student@servera ~]$ oc new-app --name=friends -i php https://github.com/swapnil-linux/php-app -e MYSQL_USER=user1 -e MYSQL_PASSWORD=redhat -e MYSQL_DATABASE=friends


--> Found image 72dffcc (6 weeks old) in image stream "openshift/php" under tag "7.3-ubi8" for "php"

    Apache 2.4 with PHP 7.3 
    ----------------------- 
    PHP 7.3 available as container is a base platform for building and running various PHP 7.3 applications and frameworks. PHP is an HTML-embedded scripting language. PHP attempts to make it easy for developers to write dynamically generated web pages. PHP also offers built-in database integration for several commercial and non-commercial database management systems, so writing a database-enabled webpage with PHP is fairly simple. The most common use of PHP coding is probably as a replacement for CGI scripts.

    Tags: builder, php, php73, php-73

    * The source repository appears to match: php
    * A source build using source code from https://github.com/swapnil-linux/php-app will be created
      * The resulting image will be pushed to image stream tag "friends:latest"
      * Use 'oc start-build' to trigger a new build

--> Creating resources ...
    imagestream.image.openshift.io "friends" created
    buildconfig.build.openshift.io "friends" created
    deployment.apps "friends" created
    service "friends" created
--> Success
    Build scheduled, use 'oc logs -f buildconfig/friends' to track its progress.
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose service/friends' 
    Run 'oc status' to view your app.
[student@servera ~]$ 

```

4\. check the build logs using oc logs command

```
[student@servera ~]$ oc logs -f buildconfig/friends
Cloning "https://github.com/swapnil-linux/php-app" ...
	Commit:	8cc735221b028c0e918bd4ca3589a716a26662bb (fixed a newline)
	Author:	Swapnil Jain <swapnil@linux.com>
	Date:	Mon Dec 14 14:17:02 2020 +1100
	.
	.
	.
	.
	.
```

5\. wait for the build to complete and create a route for friends application

```
[student@servera ~]$ oc get pods
NAME                      READY   STATUS      RESTARTS   AGE
friends-1-build           0/1     Completed   0          2m38s
friends-77fbcf885-cp7nj   1/1     Running     0          31s
mysql-5c85cb8bd7-gwt96    1/1     Running     0          4m57s


[student@servera ~]$ oc get service
NAME      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
friends   ClusterIP   172.30.45.249    <none>        8080/TCP,8443/TCP   2m42s
mysql     ClusterIP   172.30.158.133   <none>        3306/TCP            5m2s


[student@servera ~]$ oc expose service friends
route.route.openshift.io/friends exposed


[student@servera ~]$ oc get route
NAME      HOST/PORT                                             PATH   SERVICES   PORT       TERMINATION   WILDCARD
friends   friends-user15-testproject.apps.swapnil.mask365.com          friends    8080-tcp                 None


```

6\. access the application using the route from your browser

&#x20;7\. Create a table in mysql and insert few records

```
[student@servera ~]$ oc get pods 
NAME                      READY   STATUS      RESTARTS   AGE
friends-1-build           0/1     Completed   0          5m19s
friends-77fbcf885-cp7nj   1/1     Running     0          3m12s
mysql-5c85cb8bd7-gwt96    1/1     Running     0          7m38s
```

{% hint style="info" %}
note the mysql pod name
{% endhint %}

```
[student@servera ~]$ oc exec -it mysql-5c85cb8bd7-gwt96 -- bash
bash-4.2$
```

```
bash-4.2$ mysql -u$MYSQL_USER -p$MYSQL_PASSWORD friends
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

8\. refresh the application in the browser to see the results

9\. delete the application and project

```
[student@servera ~]$ oc delete all --all
pod "friends-1-build" deleted
pod "friends-2-build" deleted
pod "friends-3-build" deleted
pod "friends-69c678d6c7-6vqwj" deleted
pod "mysql-5c85cb8bd7-gwt96" deleted
service "friends" deleted
service "mysql" deleted
deployment.apps "friends" deleted
deployment.apps "mysql" deleted
buildconfig.build.openshift.io "friends" deleted
build.build.openshift.io "friends-1" deleted
build.build.openshift.io "friends-2" deleted
build.build.openshift.io "friends-3" deleted
imagestream.image.openshift.io "friends" deleted
imagestream.image.openshift.io "mysql" deleted
route.route.openshift.io "friends" deleted


[student@servera ~]$ oc get project
NAME                 DISPLAY NAME   STATUS
user15-testproject                  Active


[student@servera ~]$ oc delete project user15-testproject
project.project.openshift.io "user15-testproject" deleted
```

