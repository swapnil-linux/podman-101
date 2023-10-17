# Practice: Podman Play

1. Generate the YAML for our pod. We output our pod as a YAML definition by using podman generate kube command:

```
podman generate kube my-pod > my-pod.yaml
```

2\. Inspect the my-pod.yaml file

```
[root@servera ~]# cat my-pod.yaml
# Generation of Kubernetes YAML is still under development!
#
# Save the output of this file and use kubectl create -f to import
# it into Kubernetes.
#
# Created with podman-1.6.4
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2020-12-12T11:12:41Z"
  labels:
    app: my-pod
  name: my-pod
spec:
  containers:
  - command:
    - mysqld
    env:
    - name: PATH
      value: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    - name: TERM
      value: xterm
    - name: HOSTNAME
      value: my-pod
    - name: container
      value: podman
    - name: GOSU_VERSION
      value: "1.12"
    - name: MARIADB_VERSION
      value: 1:10.5.8+maria~focal
    - name: MYSQL_DATABASE
      value: wp
    - name: GPG_KEYS
      value: 177F4010FE56CA3336300305F1656F24C74CD1D8
    - name: MARIADB_MAJOR
      value: "10.5"
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
    resources: {}
    securityContext:
      allowPrivilegeEscalation: true
      capabilities: {}
      privileged: false
      readOnlyRootFilesystem: false
    workingDir: /
  - command:
    - apache2-foreground
    env:
    - name: PATH
      value: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    - name: TERM
      value: xterm
    - name: HOSTNAME
      value: my-pod
    - name: container
      value: podman
    - name: PHP_CPPFLAGS
      value: -fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64
    - name: WORDPRESS_VERSION
      value: "5.6"
    - name: WORDPRESS_DB_NAME
      value: wp
    - name: PHPIZE_DEPS
      value: "autoconf \t\tdpkg-dev \t\tfile \t\tg++ \t\tgcc \t\tlibc-dev \t\tmake
        \t\tpkg-config \t\tre2c"
    - name: APACHE_ENVVARS
      value: /etc/apache2/envvars
    - name: PHP_EXTRA_CONFIGURE_ARGS
      value: --with-apxs2 --disable-cgi
    - name: PHP_CFLAGS
      value: -fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64
    - name: WORDPRESS_DB_HOST
      value: 127.0.0.1
    - name: PHP_SHA256
      value: aead303e3abac23106529560547baebbedba0bb2943b91d5aa08fff1f41680f4
    - name: WORDPRESS_SHA1
      value: db8b75bfc9de27490434b365c12fd805ca6784ce
    - name: WORDPRESS_DB_USER
      value: wordpress
    - name: WORDPRESS_DB_PASSWORD
      value: w0rdpr3ss
    - name: PHP_LDFLAGS
      value: -Wl,-O1 -pie
    - name: GPG_KEYS
      value: 42670A7FE4D0441C8E4632349E4FDC074A4EF02D 5A52880781F755608BF815FC910DEB46F53EA312
    - name: PHP_URL
      value: https://www.php.net/distributions/php-7.4.13.tar.xz
    - name: PHP_EXTRA_BUILD_DEPS
      value: apache2-dev
    - name: PHP_VERSION
      value: 7.4.13
    - name: PHP_ASC_URL
      value: https://www.php.net/distributions/php-7.4.13.tar.xz.asc
    - name: PHP_INI_DIR
      value: /usr/local/etc/php
    - name: APACHE_CONFDIR
      value: /etc/apache2
    image: docker.io/library/wordpress:latest
    name: wptest-web
    resources: {}
    securityContext:
      allowPrivilegeEscalation: true
      capabilities: {}
      privileged: false
      readOnlyRootFilesystem: false
    workingDir: /var/www/html
status: {}


```

3\. clean up pods from the earlier exercise

```
[root@servera ~]# podman pod kill my-pod 
c8f626efa47ab606b1152b4c1b7e4956fa884a7a519c2b92def9698a8a9f1f96

[root@servera ~]# podman system prune --all -f --volumes 
Deleted Pods
ERRO[0000] cleanup volume (&{e7a2062141ff945d5ab3fb45bc099df815b982af661652ce3de594b9fb66b16b /var/lib/mysql [rprivate rw nodev exec nosuid rbind]}): volume e7a2062141ff945d5ab3fb45bc099df815b982af661652ce3de594b9fb66b16b is being used by the following container(s): be5f75f56c434344199b807ed7f7f46a7a17e8425389eeccbdaac946a6e39b7e: volume is being used 
ERRO[0000] cleanup volume (&{2c0446f41c8ac95b1dc14708a8111294609b4395c85ff50aafb335a8b1c60d5a /var/www/html [rprivate rw nodev exec nosuid rbind]}): volume 2c0446f41c8ac95b1dc14708a8111294609b4395c85ff50aafb335a8b1c60d5a is being used by the following container(s): caa95117fd9e78c548a7908385cf98c7c70b6fa9abf50401f30f69d8485d85a6: volume is being used 
c8f626efa47ab606b1152b4c1b7e4956fa884a7a519c2b92def9698a8a9f1f96
Deleted Containers
Deleted Volumes
2c0446f41c8ac95b1dc14708a8111294609b4395c85ff50aafb335a8b1c60d5a
e7a2062141ff945d5ab3fb45bc099df815b982af661652ce3de594b9fb66b16b
Deleted Images
da86e6ba6ca197bf6bc5e9d900febd906b133eaa4750e6bed647b0fbe50ed43e
3a348a04a8159339ed3ca053ea925f854252e6a6c3df6fa82c17625d1026f18b
bc5f6567b76345eb23d6d0c13a9c4540d9cf754ac536d406d9dbc341147c6c77


[root@servera ~]# podman ps -a --pod
CONTAINER ID  IMAGE  COMMAND  CREATED  STATUS  PORTS  NAMES  POD
```

4\. Next, use `podman play kube` to start your pod from the defined YAML:

```
[root@servera ~]# podman play kube my-pod.yaml 


Trying to pull docker.io/library/mariadb:latest...
Getting image source signatures
Copying blob 22776aa82430 done  
Copying blob 14428a6d4bcd done  
Copying blob 2c2d948710f2 done  
Copying blob 90e64230d63d done  
Copying blob da7391352a9b done  
Copying blob f30861f14a10 done  
Copying blob 420a23f08c41 done  
Copying blob e8e9e6a3da24 done  
Copying blob a8690a3260b7 done  
Copying blob bd73f23de482 done  
Copying blob 4202ba90333a done  
Copying blob a33f860b4aa6 done  
Copying config 3a348a04a8 done  
Writing manifest to image destination
Storing signatures
Trying to pull docker.io/library/wordpress:latest...
Getting image source signatures
Copying blob 6ec7b7d162b2 done  
Copying blob db606474d60c done  
Copying blob 4c761b44e2cc done  
Copying blob 3bb2e8051594 done  
Copying blob afb30f0cd8e0 done  
Copying blob c2199db96575 done  
Copying blob 1b9a9381eea8 done  
Copying blob 50450ffc67ee done  
Copying blob 4d1e5a768e83 done  
Copying blob 5e8be0d1df16 done  
Copying blob 7a6395859d40 done  
Copying blob 7306499d3dce done  
Copying blob fa6f0ba15ac6 done  
Copying blob 308a9ead128f done  
Copying blob a08dd591ed8a done  
Copying blob 2db781a8732e done  
Copying blob 63d3161e9e46 done  
Copying blob 931a26282f2a done  
Copying blob caf2bb847f73 done  
Copying blob f5c6b405e809 done  
Copying config bc5f6567b7 done  
Writing manifest to image destination
Storing signatures
Pod:
19e69353543a1b7826d230e144fb1b5c96c488916af377029e791934c897ba5e
Containers:
fd8081a02df024d3b336ec0c34681ab8549205afa235bd800bb1cebd834cd84c
311c6c9603a3670d19cd458b6a24fbf62807e0e2ff8b638b826cb78e725ea2f2
[root@servera ~]# 
```

```
[root@servera ~]# podman pod ps
POD ID         NAME     STATUS    CREATED         # OF CONTAINERS   INFRA ID
19e69353543a   my-pod   Running   2 minutes ago   3                 d72c6fc858d0
[root@servera ~]# 

[root@servera ~]# podman ps -a --pod
CONTAINER ID  IMAGE                               COMMAND               CREATED             STATUS                 PORTS                  NAMES               POD
311c6c9603a3  docker.io/library/wordpress:latest  docker-entrypoint...  About a minute ago  Up About a minute ago  0.0.0.0:38080->80/tcp  wptest-web          19e69353543a
fd8081a02df0  docker.io/library/mariadb:latest    docker-entrypoint...  2 minutes ago       Up About a minute ago  0.0.0.0:38080->80/tcp  wptest-db           19e69353543a
d72c6fc858d0  k8s.gcr.io/pause:3.1                                      2 minutes ago       Up About a minute ago  0.0.0.0:38080->80/tcp  19e69353543a-infra  19e69353543a
[root@servera ~]# 
```

5\. also notice the namespaces that are being shared

```
[root@servera ~]# podman ps --ns
CONTAINER ID  NAMES       PID   CGROUPNS  IPC         MNT         NET         PIDNS       USERNS      UTS
311c6c9603a3  wptest-web  8643            4026532225  4026532229  4026532142  4026532230  4026531837  4026532224
fd8081a02df0  wptest-db   8611            4026532225  4026532227  4026532142  4026532228  4026531837  4026532224
```

6\. You should be able to reach the wordpress setup page using your system IP or name and port 38080, just like before.

![](<../.gitbook/assets/Screen Shot 2020-12-12 at 10.21.56 pm.png>)
