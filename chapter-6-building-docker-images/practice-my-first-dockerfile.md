# Practice: My First Containerfile

1: Create a working directory as myhttpd

```
$ mkdir myhttpd
$ cd myhttpd
```

2\.  Create a file named Containerfile with the below content

```
FROM registry.access.redhat.com/ubi9/ubi:latest
RUN dnf update -y && dnf install httpd -y && dnf clean all
LABEL Description="My First Containerfile"
CMD ["/usr/sbin/httpd","-DFOREGROUND"]
```

3\. Build the image using podman build command

```
$ podman build -t myhttpd .

STEP 1/4: FROM registry.access.redhat.com/ubi9/ubi:latest
Trying to pull registry.access.redhat.com/ubi9/ubi:latest...
Getting image source signatures
Checking if image destination supports signatures
Copying blob a3ed95caeb02 done   |
Copying blob 7aab3b38016c done   |
Copying config 5eb5554382 done   |
Writing manifest to image destination
Storing signatures
STEP 2/4: RUN dnf update -y && dnf install httpd -y && dnf clean all
...
...
STEP 3/4: LABEL Description="My First Containerfile"
STEP 4/4: CMD ["/usr/sbin/httpd","-DFOREGROUND"]
COMMIT myhttpd
--> fbcbf6665a3f
fbcbf6665a3f7ad6ad444fae7fda4688a3157c30b0ce62e168d4b99992704dea
```

```
$ podman images

REPOSITORY                            TAG      IMAGE ID       CREATED              SIZE
localhost/myhttpd                     latest   fbcbf6665a3f   About a minute ago   290 MB
registry.access.redhat.com/ubi9/ubi   latest   5eb5554382e4   2 weeks ago          217 MB
```

4\. Test the image using podman run

```
$ podman run --name=web1 -p 38080:80 -d myhttpd
385c9b9885a0d012216b628c74b93292f5a14393a31182bbc25f6797bbd316e8


$ podman ps
CONTAINER ID  IMAGE                     COMMAND               CREATED        STATUS            PORTS                  NAMES
385c9b9885a0  localhost/myhttpd:latest  /usr/sbin/httpd -...  2 seconds ago  Up 2 seconds ago  0.0.0.0:38080->80/tcp  web1

```

Check if you are able to connect to the webserver on port 38080

5\. Clean up

```
podman kill web1

podman rm web1
```

_**Congratulations! this concludes the exercise.**_
