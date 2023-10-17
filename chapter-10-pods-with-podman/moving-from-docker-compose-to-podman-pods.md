# Moving from docker-compose to Podman pods

Podman does not have a counterpart to the `docker-compose` command. Well, it does, sort of. There's a project in the works called [podman-compose](https://github.com/containers/podman-compose), which is supposed to do the same basic thing as `docker-compose`. It is still in its emerging stage.

### Podman play

Podman does, however, let you import Kubernetes definitions using the `podman play` command. Kubernetes definitions are YAML. And sounds like a docker-compose alternative

```
[root@servera ~]# podman play --help
Play a pod

Description:
  Play a pod and its containers from a structured file.

Usage:
  podman play [command]

Available Commands:
  kube        Play a pod based on Kubernetes YAML


```

### Podman generate

Podman lets you generate Kubernetes definitions from the existing runtime. For example, if you have a running container, you can use `podman generate` to create a YAML file to define that container.&#x20;

```
[root@servera ~]# podman generate --help
Generate structured data based for a containers and pods

Usage:
  podman generate [command]

Available Commands:
  kube        Generate Kubernetes pod YAML from a container or pod
  systemd     Generate a systemd unit file for a Podman container

[root@servera ~]# 
```
