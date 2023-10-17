# Practice: Installing and testing Podman

## CentOS

Podman is available in the default Extras repos for CentOS 7 and in the AppStream repo for CentOS 8 and Stream. Even though the available version often lags behind the latest upstream release, it’s still the preferable build for production environments.

```
yum -y install podman
```

The [Kubic project](https://build.opensuse.org/project/show/devel:kubic:libcontainers:stable) provides updated packages for CentOS 7, 8 and Stream. These packages haven’t been through the official Red Hat QA process and may not be preferable for production environments.

```
# CentOS 7
sudo curl -L -o /etc/yum.repos.d/devel:kubic:libcontainers:stable.repo https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/CentOS_7/devel:kubic:libcontainers:stable.repo
sudo yum -y install podman

# CentOS 8
sudo dnf -y module disable container-tools
sudo dnf -y install 'dnf-command(copr)'
sudo dnf -y copr enable rhcontainerbot/container-selinux
sudo curl -L -o /etc/yum.repos.d/devel:kubic:libcontainers:stable.repo https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/CentOS_8/devel:kubic:libcontainers:stable.repo
sudo dnf -y install podman
sudo dnf -y update
```

## RHEL 7

Subscribe, then enable Extras channel and install Podman.

```
sudo subscription-manager repos --enable=rhel-7-server-extras-rpms
sudo yum -y install podman
```

## RHEL 8

Enable PowerTools repo and install Podman

```
dnf config-manager --set-enabled PowerTools

dnf install -y @container-tools
```

## Ubuntu

The libpod package is available in the official repositories for Ubuntu 20.10 and newer.

```
# Ubuntu 20.10 and newer
sudo apt-get -y update
sudo apt-get -y install podman
```

## Testing Podman

1. Check podman version

```
[root@servera ~]# podman version
Version:            1.6.4
RemoteAPI Version:  1
Go Version:         go1.12.12
OS/Arch:            linux/amd64
```

2\. pull alpine image using podman pull command

```
[root@servera ~]# podman pull docker.io/alpine
Trying to pull docker.io/alpine...
Getting image source signatures
Copying blob 188c0c94c7c5 done  
Copying config d6e46aa247 done  
Writing manifest to image destination
Storing signatures
d6e46aa2470df1d32034c6707c8041158b652f38d2a9ae3d7ad7e7532d22ebe0
```

3\. Create a test container using podman run command

```
podman run --name test-container-1 docker.io/alpine echo Hello World
```

4\. list the container using podman ps command

```
[root@servera ~]# podman ps -a
CONTAINER ID  IMAGE                            COMMAND                CREATED         STATUS                    PORTS  NAMES
c23ccdbcd596  docker.io/library/alpine:latest  echo Hello World       17 seconds ago  Exited (0) 4 seconds ago         test-container-1
```

5\. remove the container and image

```
[root@servera ~]# podman rm test-container-1
c23ccdbcd596ae484a865d922f30f4de8523ef6bbe76995f4312e181435d2459
```

```
[root@servera ~]# podman rmi docker.io/alpine
Untagged: docker.io/library/alpine:latest
Deleted: d6e46aa2470df1d32034c6707c8041158b652f38d2a9ae3d7ad7e7532d22ebe0
```

_**Congratulations! you have successfully installed and tested podman.**_

