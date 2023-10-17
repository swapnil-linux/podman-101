# Practice: Working with images

1\. Search for nginx image and limit the results to only 3

```
[root@servera ~]# podman search nginx --limit 3
INDEX        NAME                                               DESCRIPTION                                       STARS   OFFICIAL   AUTOMATED
redhat.com   registry.access.redhat.com/rhscl/nginx-112-rhel7   Nginx is a web server and a reverse proxy se...   0                  
redhat.com   registry.access.redhat.com/ubi8/nginx-118          Platform for running nginx 1.18 or building ...   0                  
redhat.com   registry.access.redhat.com/rhscl/nginx-18-rhel7    Nginx 1.8 server and a reverse proxy server       0                  
redhat.io    registry.redhat.io/rhel8/nginx-116                 Platform for running nginx 1.16 or building ...   0                  
redhat.io    registry.redhat.io/rhscl/nginx-112-rhel7           Nginx is a web server and a reverse proxy se...   0                  
redhat.io    registry.redhat.io/rhel8/nginx-114                 Nginx is a web server and a reverse proxy se...   0                  
docker.io    docker.io/library/nginx                            Official build of Nginx.                          14133   [OK]       
docker.io    docker.io/jwilder/nginx-proxy                      Automated Nginx reverse proxy for docker con...   1921               [OK]
docker.io    docker.io/bitnami/nginx                            Bitnami nginx Docker Image                        91                 [OK]
[root@servera ~]# 

```

2\. add quay.io to the registries in /etc/containers/registries.conf and do the search again

```
[root@servera ~]# grep ^registries /etc/containers/registries.conf 
registries = ['registry.access.redhat.com', 'registry.redhat.io', 'docker.io', 'quay.io']

```

```
[root@servera ~]# podman search nginx --limit 3
INDEX        NAME                                                             DESCRIPTION                                       STARS   OFFICIAL   AUTOMATED
redhat.com   registry.access.redhat.com/rhscl/nginx-112-rhel7                 Nginx is a web server and a reverse proxy se...   0                  
redhat.com   registry.access.redhat.com/ubi8/nginx-118                        Platform for running nginx 1.18 or building ...   0                  
redhat.com   registry.access.redhat.com/rhscl/nginx-18-rhel7                  Nginx 1.8 server and a reverse proxy server       0                  
redhat.io    registry.redhat.io/rhel8/nginx-116                               Platform for running nginx 1.16 or building ...   0                  
redhat.io    registry.redhat.io/rhscl/nginx-112-rhel7                         Nginx is a web server and a reverse proxy se...   0                  
redhat.io    registry.redhat.io/ubi8/nginx-118                                Platform for running nginx 1.18 or building ...   0                  
docker.io    docker.io/library/nginx                                          Official build of Nginx.                          14133   [OK]       
docker.io    docker.io/jwilder/nginx-proxy                                    Automated Nginx reverse proxy for docker con...   1921               [OK]
docker.io    docker.io/bitnami/nginx                                          Bitnami nginx Docker Image                        91                 [OK]
quay.io      quay.io/kubernetes-ingress-controller/nginx-ingress-controller   NGINX Ingress controller built around the [K...   0                  
quay.io      quay.io/opencloudio/ibm-ingress-nginx-operator                   # IBM Ingress Nginx Operator  **Important:**...   0                  
quay.io      quay.io/ukhomeofficedigital/nginx-proxy                          # OpenResty Docker Container  [![Build Statu...   0      
```

3\. Change the search order in registries.conf giving quay.io first priority. And, do the search again.

```
[root@servera ~]# grep ^registries /etc/containers/registries.conf 
registries = ['quay.io','registry.access.redhat.com', 'registry.redhat.io', 'docker.io']
```

```
[root@servera ~]# podman search nginx --limit 3
INDEX        NAME                                                             DESCRIPTION                                       STARS   OFFICIAL   AUTOMATED
quay.io      quay.io/kubernetes-ingress-controller/nginx-ingress-controller   NGINX Ingress controller built around the [K...   0                  
quay.io      quay.io/opencloudio/ibm-ingress-nginx-operator                   # IBM Ingress Nginx Operator  **Important:**...   0                  
quay.io      quay.io/ukhomeofficedigital/nginx-proxy                          # OpenResty Docker Container  [![Build Statu...   0                  
redhat.com   registry.access.redhat.com/rhscl/nginx-112-rhel7                 Nginx is a web server and a reverse proxy se...   0                  
redhat.com   registry.access.redhat.com/ubi8/nginx-118                        Platform for running nginx 1.18 or building ...   0                  
redhat.com   registry.access.redhat.com/rhscl/nginx-18-rhel7                  Nginx 1.8 server and a reverse proxy server       0                  
redhat.io    registry.redhat.io/rhel8/nginx-116                               Platform for running nginx 1.16 or building ...   0                  
redhat.io    registry.redhat.io/rhscl/nginx-112-rhel7                         Nginx is a web server and a reverse proxy se...   0                  
redhat.io    registry.redhat.io/ubi8/nginx-118                                Platform for running nginx 1.18 or building ...   0                  
docker.io    docker.io/library/nginx                                          Official build of Nginx.                          14133   [OK]       
docker.io    docker.io/jwilder/nginx-proxy                                    Automated Nginx reverse proxy for docker con...   1921               [OK]
docker.io    docker.io/bitnami/nginx                                          Bitnami nginx Docker Image                        91                 [OK]
[root@servera ~]# 

```

4\. Pull the nginx image without specifying the index.

```
[root@servera ~]# podman pull nginx
Trying to pull quay.io/nginx...
  error parsing HTTP 404 response body: invalid character '<' looking for beginning of value: "<!DOCTYPE HTML PUBLIC \"-//W3C//DTD HTML 3.2 Final//EN\">\n<title>404 Not Found</title>\n<h1>Not Found</h1>\n<p>The requested URL was not found on the server. If you entered the URL manually please check your spelling and try again.</p>\n"
Trying to pull registry.access.redhat.com/nginx...
  unsupported: This repo requires terms acceptance and is only available on registry.redhat.io
Trying to pull registry.redhat.io/nginx...
  unable to retrieve auth token: invalid username/password: unauthorized: Please login to the Red Hat Registry using your Customer Portal credentials. Further instructions can be found here: https://access.redhat.com/RegistryAuthentication
Trying to pull docker.io/library/nginx...
Getting image source signatures
Copying blob 852e50cd189d skipped: already exists  
Copying blob 8b03f1e11359 done  
Copying blob addb10abd9cb done  
Copying blob 571d7e852307 done  
Copying blob d20aa7ccdb77 done  
Copying config bc9a0695f5 done  
Writing manifest to image destination
Storing signatures
bc9a0695f5712dcaaa09a5adc415a3936ccba13fc2587dfd76b1b8aeea3f221c
[root@servera ~]# 

```

5\. Pull another ngnix image with stable-alpine tag

```
[root@servera ~]# podman pull nginx:stable-alpine
Trying to pull quay.io/nginx:stable-alpine...
  error parsing HTTP 404 response body: invalid character '<' looking for beginning of value: "<!DOCTYPE HTML PUBLIC \"-//W3C//DTD HTML 3.2 Final//EN\">\n<title>404 Not Found</title>\n<h1>Not Found</h1>\n<p>The requested URL was not found on the server. If you entered the URL manually please check your spelling and try again.</p>\n"
Trying to pull registry.access.redhat.com/nginx:stable-alpine...
  unsupported: This repo requires terms acceptance and is only available on registry.redhat.io
Trying to pull registry.redhat.io/nginx:stable-alpine...
  unable to retrieve auth token: invalid username/password: unauthorized: Please login to the Red Hat Registry using your Customer Portal credentials. Further instructions can be found here: https://access.redhat.com/RegistryAuthentication
Trying to pull docker.io/library/nginx:stable-alpine...
Getting image source signatures
Copying blob 6461dd5d0077 done  
Copying blob cbdbe7a5bc2a done  
Copying blob 70e031ddac33 done  
Copying blob 028906aee9a5 done  
Copying blob 656e57b81606 done  
Copying config a8054a07a2 done  
Writing manifest to image destination
Storing signatures
a8054a07a2ce2b3b6d809f05b09a9b0187ccca47c09821b0b474cf29b378c3d9
[root@servera ~]# 


```

6\. List all images from nginx repository and create a container using the nginx image with stable-alpine tag

```
[root@servera ~]# podman images docker.io/library/nginx
REPOSITORY                TAG             IMAGE ID       CREATED       SIZE
docker.io/library/nginx   stable-alpine   a8054a07a2ce   2 weeks ago   23.4 MB
docker.io/library/nginx   latest          bc9a0695f571   2 weeks ago   137 MB
[root@servera ~]# 

```

```
[root@servera ~]# podman run --name web-con -d docker.io/library/nginx:stable-alpine 
bd674ab76524664f61250b77efe96cc57e48c056d1bd99904f48c2e914d8cca0

[root@servera ~]# podman ps
CONTAINER ID  IMAGE                                  COMMAND               CREATED        STATUS            PORTS  NAMES
bd674ab76524  docker.io/library/nginx:stable-alpine  nginx -g daemon o...  4 seconds ago  Up 4 seconds ago         web-con
 
```

7\. Try deleting all images

```
[root@servera ~]# podman rmi docker.io/library/nginx:latest docker.io/library/nginx:stable-alpine 
Untagged: docker.io/library/nginx:latest
Deleted: bc9a0695f5712dcaaa09a5adc415a3936ccba13fc2587dfd76b1b8aeea3f221c
Error: could not remove image a8054a07a2ce2b3b6d809f05b09a9b0187ccca47c09821b0b474cf29b378c3d9 as it is being used by 1 containers


[root@servera ~]# podman images docker.io/library/nginx
REPOSITORY                TAG             IMAGE ID       CREATED       SIZE
docker.io/library/nginx   stable-alpine   a8054a07a2ce   2 weeks ago   23.4 MB

```

8\. Stop and remove the web-con contaner,  and try deleting image again.

```
[root@servera ~]# podman stop web-con
bd674ab76524664f61250b77efe96cc57e48c056d1bd99904f48c2e914d8cca0

[root@servera ~]# podman rm web-con
bd674ab76524664f61250b77efe96cc57e48c056d1bd99904f48c2e914d8cca0

[root@servera ~]# podman rmi docker.io/library/nginx:stable-alpine 
Untagged: docker.io/library/nginx:stable-alpine
Deleted: a8054a07a2ce2b3b6d809f05b09a9b0187ccca47c09821b0b474cf29b378c3d9

[root@servera ~]# podman images
REPOSITORY   TAG   IMAGE ID   CREATED   SIZE

```

_**Congratulations! this concludes the exercise.**_
