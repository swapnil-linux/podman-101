# Practice: Podman Kube Play

1. Generate the YAML for our pod. We output our pod as a YAML definition by using the `podman kube generate` command:

```
podman kube generate my-pod > my-pod.yaml
```

2\. Inspect the my-pod.yaml file

```
$ cat my-pod.yaml
# Save the output of this file and use kubectl create -f to import
# it into Kubernetes.
#
# Created with podman-5.4.2
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2025-03-15T11:12:41Z"
  labels:
    app: my-pod
  name: my-pod
spec:
  containers:
  - command:
    - mariadbd
    env:
    - name: MYSQL_DATABASE
      value: wp
    - name: MYSQL_ROOT_PASSWORD
      value: myrootpass
    - name: MYSQL_USER
      value: wordpress
    - name: MYSQL_PASSWORD
      value: w0rdpr3ss
    image: docker.io/library/mariadb:latest
    name: wptest-db
    ports:
    - containerPort: 80
      hostPort: 38080
      protocol: TCP
  - command:
    - apache2-foreground
    env:
    - name: WORDPRESS_DB_NAME
      value: wp
    - name: WORDPRESS_DB_HOST
      value: 127.0.0.1
    - name: WORDPRESS_DB_USER
      value: wordpress
    - name: WORDPRESS_DB_PASSWORD
      value: w0rdpr3ss
    image: docker.io/library/wordpress:latest
    name: wptest-web
status: {}


```

3\. Clean up pods from the earlier exercise

```
$ podman pod kill my-pod 
my-pod

$ podman system prune --all -f --volumes 

$ podman ps -a --pod
CONTAINER ID  IMAGE  COMMAND  CREATED  STATUS  PORTS  NAMES  POD
```

4\. Next, use `podman kube play` to start your pod from the defined YAML:

```
$ podman kube play my-pod.yaml 

Trying to pull docker.io/library/mariadb:latest...
Getting image source signatures
Copying blob 22776aa82430 done   |
...
Writing manifest to image destination
Trying to pull docker.io/library/wordpress:latest...
Getting image source signatures
Copying blob 6ec7b7d162b2 done   |
...
Writing manifest to image destination
Pod:
19e69353543a1b7826d230e144fb1b5c96c488916af377029e791934c897ba5e
Containers:
fd8081a02df024d3b336ec0c34681ab8549205afa235bd800bb1cebd834cd84c
311c6c9603a3670d19cd458b6a24fbf62807e0e2ff8b638b826cb78e725ea2f2
```

```
$ podman pod ps
POD ID         NAME     STATUS    CREATED         # OF CONTAINERS   INFRA ID
19e69353543a   my-pod   Running   2 minutes ago   3                 d72c6fc858d0

$ podman ps -a --pod
CONTAINER ID  IMAGE                               COMMAND               CREATED             STATUS                 PORTS                  NAMES               POD
311c6c9603a3  docker.io/library/wordpress:latest  apache2-foregroun...  About a minute ago  Up About a minute ago  0.0.0.0:38080->80/tcp  my-pod-wptest-web   19e69353543a
fd8081a02df0  docker.io/library/mariadb:latest    mariadbd              2 minutes ago       Up About a minute ago  0.0.0.0:38080->80/tcp  my-pod-wptest-db    19e69353543a
d72c6fc858d0  localhost/podman-pause:5.4.2-...                          2 minutes ago       Up About a minute ago  0.0.0.0:38080->80/tcp  19e69353543a-infra  19e69353543a
```

5\. Also notice the namespaces that are being shared

```
$ podman ps --ns
CONTAINER ID  NAMES              PID   CGROUPNS  IPC         MNT         NET         PIDNS       USERNS      UTS
311c6c9603a3  my-pod-wptest-web  8643            4026532225  4026532229  4026532142  4026532230  4026531837  4026532224
fd8081a02df0  my-pod-wptest-db   8611            4026532225  4026532227  4026532142  4026532228  4026531837  4026532224
```

6\. You should be able to reach the WordPress setup page using your system IP or name and port 38080, just like before.

![](<../.gitbook/assets/Screen Shot 2020-12-12 at 10.21.56 pm.png>)

7\. To tear down the pod created from YAML, use:

```
podman kube down my-pod.yaml
```
