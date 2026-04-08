# Practice: Working with images

1\. Search for nginx image and limit the results to only 3

```
$ podman search nginx --limit 3
INDEX        NAME                                               DESCRIPTION                                       STARS   OFFICIAL   AUTOMATED
redhat.com   registry.access.redhat.com/ubi9/nginx-124          Platform for running nginx 1.24 or building ...   0                  
redhat.com   registry.access.redhat.com/ubi9/nginx-122          Platform for running nginx 1.22 or building ...   0                  
redhat.com   registry.access.redhat.com/ubi8/nginx-120          Platform for running nginx 1.20 or building ...   0                  
docker.io    docker.io/library/nginx                            Official build of Nginx.                          20133   [OK]       
docker.io    docker.io/bitnami/nginx                            Bitnami nginx Docker Image                        191                [OK]
docker.io    docker.io/nginxinc/nginx-unprivileged              Unprivileged NGINX Dockerfiles                    151
```

2\. Add quay.io to the unqualified search registries in /etc/containers/registries.conf and do the search again

```toml
# /etc/containers/registries.conf
unqualified-search-registries = ["registry.access.redhat.com", "registry.redhat.io", "docker.io", "quay.io"]
```

```
$ podman search nginx --limit 3
INDEX        NAME                                                             DESCRIPTION                                       STARS   OFFICIAL   AUTOMATED
redhat.com   registry.access.redhat.com/ubi9/nginx-124                        Platform for running nginx 1.24 or building ...   0                  
redhat.com   registry.access.redhat.com/ubi9/nginx-122                        Platform for running nginx 1.22 or building ...   0                  
redhat.com   registry.access.redhat.com/ubi8/nginx-120                        Platform for running nginx 1.20 or building ...   0                  
docker.io    docker.io/library/nginx                                          Official build of Nginx.                          20133   [OK]       
docker.io    docker.io/bitnami/nginx                                          Bitnami nginx Docker Image                        191                [OK]
docker.io    docker.io/nginxinc/nginx-unprivileged                            Unprivileged NGINX Dockerfiles                    151
quay.io      quay.io/kubernetes-ingress-controller/nginx-ingress-controller   NGINX Ingress controller built around the [K...   0                  
quay.io      quay.io/nginx/nginx-ingress                                      NGINX Ingress Controller for Kubernetes          0                  
quay.io      quay.io/ukhomeofficedigital/nginx-proxy                          OpenResty Docker Container                        0      
```

3\. Change the search order in registries.conf giving quay.io first priority. And, do the search again.

```toml
unqualified-search-registries = ["quay.io", "registry.access.redhat.com", "registry.redhat.io", "docker.io"]
```

4\. Pull the nginx image using the fully-qualified name.

```
$ podman pull docker.io/library/nginx
Trying to pull docker.io/library/nginx:latest...
Getting image source signatures
Copying blob 8b03f1e11359 done   |
Copying blob addb10abd9cb done   |
Copying blob 571d7e852307 done   |
Copying blob d20aa7ccdb77 done   |
Copying config bc9a0695f5 done   |
Writing manifest to image destination
bc9a0695f5712dcaaa09a5adc415a3936ccba13fc2587dfd76b1b8aeea3f221c

```

5\. Pull another nginx image with alpine tag

```
$ podman pull docker.io/library/nginx:alpine
Trying to pull docker.io/library/nginx:alpine...
Getting image source signatures
Copying blob 6461dd5d0077 done   |
Copying blob cbdbe7a5bc2a done   |
Copying blob 70e031ddac33 done   |
Copying blob 028906aee9a5 done   |
Copying config a8054a07a2 done   |
Writing manifest to image destination
a8054a07a2ce2b3b6d809f05b09a9b0187ccca47c09821b0b474cf29b378c3d9

```

6\. List all images from nginx repository and create a container using the nginx image with alpine tag

```
$ podman images docker.io/library/nginx
REPOSITORY                TAG      IMAGE ID       CREATED       SIZE
docker.io/library/nginx   alpine   a8054a07a2ce   2 weeks ago   43.3 MB
docker.io/library/nginx   latest   bc9a0695f571   2 weeks ago   192 MB

```

```
$ podman run --name web-con -d docker.io/library/nginx:alpine 
bd674ab76524664f61250b77efe96cc57e48c056d1bd99904f48c2e914d8cca0

$ podman ps
CONTAINER ID  IMAGE                                COMMAND               CREATED        STATUS            PORTS  NAMES
bd674ab76524  docker.io/library/nginx:alpine       nginx -g daemon o...  4 seconds ago  Up 4 seconds ago         web-con
 
```

7\. Try deleting all images

```
$ podman rmi docker.io/library/nginx:latest docker.io/library/nginx:alpine 
Untagged: docker.io/library/nginx:latest
Deleted: bc9a0695f5712dcaaa09a5adc415a3936ccba13fc2587dfd76b1b8aeea3f221c
Error: could not remove image a8054a07a2ce as it is being used by 1 containers


$ podman images docker.io/library/nginx
REPOSITORY                TAG      IMAGE ID       CREATED       SIZE
docker.io/library/nginx   alpine   a8054a07a2ce   2 weeks ago   43.3 MB

```

8\. Stop and remove the web-con container, and try deleting image again.

```
$ podman stop web-con
bd674ab76524

$ podman rm web-con
bd674ab76524

$ podman rmi docker.io/library/nginx:alpine 
Untagged: docker.io/library/nginx:alpine
Deleted: a8054a07a2ce2b3b6d809f05b09a9b0187ccca47c09821b0b474cf29b378c3d9

$ podman images
REPOSITORY   TAG   IMAGE ID   CREATED   SIZE

```

_**Congratulations! this concludes the exercise.**_
