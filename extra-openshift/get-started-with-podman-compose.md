# Get started with Podman Compose

Podman now includes a built-in `podman compose` subcommand that delegates to an external compose provider while using Podman as the container engine. The two main compose providers are:

* [podman-compose](https://github.com/containers/podman-compose) — a Python-based tool that translates Compose files into Podman commands
* [docker-compose](https://github.com/docker/compose) — the official Docker Compose tool, which works with Podman via its Docker-compatible API socket

The `podman compose` command automatically detects which provider is installed and uses it.

### Installing podman-compose

```
# Using pip
pip3 install podman-compose

# On Fedora/RHEL
sudo dnf install podman-compose
```

### Using podman compose

1\. Create a working directory and a `docker-compose.yml` (or `compose.yaml`) file

```
mkdir myapp
cd myapp
```

2\. Create a `compose.yaml` file. Here's an example with a web app and Redis:

```yaml
services:
  web:
    image: docker.io/library/nginx
    ports:
      - "8080:80"
  redis:
    image: docker.io/library/redis:alpine
```

3\. Bring up the services

```
$ podman compose up -d
>>>> Executing external compose provider...
Creating pod myapp
Creating container myapp_web_1
Creating container myapp_redis_1

```

4\. Check the status

```
$ podman compose ps
CONTAINER ID  IMAGE                            COMMAND               STATUS            PORTS                 NAMES
a1b2c3d4e5f6  docker.io/library/nginx:latest   nginx -g daemon o...  Up 30 seconds     0.0.0.0:8080->80/tcp  myapp_web_1
f6e5d4c3b2a1  docker.io/library/redis:alpine   redis-server          Up 30 seconds                           myapp_redis_1
```

5\. Access the application via the exposed port (http://localhost:8080)

6\. Tear down

```
$ podman compose down
```

{% hint style="info" %}
`podman compose` works with standard `docker-compose.yml` / `compose.yaml` files, making migration from Docker straightforward. It runs rootless by default.
{% endhint %}
