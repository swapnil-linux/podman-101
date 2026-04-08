# Moving from docker-compose to Podman pods

### Podman Compose

Podman now includes a built-in `podman compose` subcommand that works with standard Docker Compose files. It delegates to an external compose provider — either [docker-compose](https://github.com/docker/compose) or [podman-compose](https://github.com/containers/podman-compose) — while using Podman as the container engine.

```
# Using podman compose with a docker-compose.yml file
podman compose up -d
podman compose ps
podman compose down
```

{% hint style="info" %}
If you have `docker-compose` or `podman-compose` installed, `podman compose` will automatically detect and use it. Install one of them with:
`pip install podman-compose` or `dnf install podman-compose`
{% endhint %}

### Podman Kube Play

Podman lets you import Kubernetes YAML definitions using the `podman kube play` command. This provides a Kubernetes-native alternative to docker-compose:

```
$ podman kube play --help
Play a pod or volume based on Kubernetes YAML

Description:
  Reads in a structured file of Kubernetes YAML.
  Creates pods or volumes based on the Kubernetes kind described in the YAML.

Usage:
  podman kube play [options] KUBEFILE [flags]
```

### Podman Kube Generate

Podman lets you generate Kubernetes YAML definitions from the existing runtime. For example, if you have a running container or pod, you can use `podman kube generate` to create a YAML file to define it.&#x20;

```
$ podman kube generate --help
Generate Kubernetes YAML from containers, pods or volumes

Usage:
  podman kube generate [options] {CONTAINER...|POD...|VOLUME...} [flags]
```

### Quadlet — Systemd Integration

Podman also supports **Quadlet**, a way to run containers and pods as systemd services. Quadlet files are placed in `/etc/containers/systemd/` (for root) or `~/.config/containers/systemd/` (for rootless) and describe containers, pods, networks, and volumes using a systemd-like syntax.

Example Quadlet container file (`~/.config/containers/systemd/myapp.container`):

```ini
[Container]
Image=docker.io/library/nginx
PublishPort=8080:80

[Service]
Restart=always

[Install]
WantedBy=default.target
```

After creating the file, reload systemd and start the service:

```
systemctl --user daemon-reload
systemctl --user start myapp
```

Quadlet is the recommended approach for running containers as system services, replacing the older `podman generate systemd` command.
