# Practice: My First Dockerfile

1: Create a working directory as myhttpd

```
[root@servera ~]# mkdir myhttpd
[root@servera ~]# cd myhttpd
[root@servera myhttpd]# 
```

2\.  create a file named Dockerfile with the below content

```
FROM registry.access.redhat.com/ubi7/ubi:latest
RUN yum update -y && yum install httpd -y && yum clean all
LABEL Description My First Dockerfile
CMD ["/usr/sbin/httpd","-DFOREGROUND"]
```

3\. build the image using podman build command

```
[root@servera myhttpd]# podman build -t myhttpd .


STEP 1: FROM registry.access.redhat.com/ubi7/ubi:latest
Getting image source signatures
Copying blob d518f7038c56 done  
Copying blob 2b63304f1869 done  
Copying config cfd6d0c09e done  
Writing manifest to image destination
Storing signatures
STEP 2: RUN yum update -y && yum install httpd -y
...
...
...
...
6ab10af4b43e45f7960dc6a988688fea8e0fd308668c08c10cebff17a0bbbbe7
STEP 3: LABEL Description My First Dockerfile
516c593639061278f1548dea6c68797c299a7fb3e57622f5b1cef68f4764ed87
STEP 4: CMD ["/usr/sbin/httpd","-DFOREGROUND"]
STEP 5: COMMIT myhttpd
fbcbf6665a3f7ad6ad444fae7fda4688a3157c30b0ce62e168d4b99992704dea
fbcbf6665a3f7ad6ad444fae7fda4688a3157c30b0ce62e168d4b99992704dea
```

```
[root@servera myhttpd]# podman images


REPOSITORY                            TAG      IMAGE ID       CREATED              SIZE
localhost/myhttpd                     latest   fbcbf6665a3f   About a minute ago   255 MB
registry.access.redhat.com/ubi7/ubi   latest   cfd6d0c09e95   5 weeks ago          216 MB
[root@servera myhttpd]# 
```

4\. Test the image using podman run

```
[root@servera myhttpd]# podman run --name=web1 -p 38080:80 -d myhttpd
385c9b9885a0d012216b628c74b93292f5a14393a31182bbc25f6797bbd316e8


[root@servera myhttpd]# podman ps
CONTAINER ID  IMAGE                     COMMAND               CREATED        STATUS            PORTS                  NAMES
385c9b9885a0  localhost/myhttpd:latest  /usr/sbin/httpd -...  2 seconds ago  Up 2 seconds ago  0.0.0.0:38080->80/tcp  web1






```

check if you able to connect to the webserver on port 38080

5\. Clean up

```
podman kill web1

podman rm web1
```

_**Congratulations! this concludes the exercise.**_

