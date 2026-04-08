# Podman

[Podman](https://podman.io/) is a daemonless, open source tool designed to make it easy to find, run, build, share and deploy applications using Open Containers Initiative ([OCI](https://www.opencontainers.org/)) [Containers](https://developers.redhat.com/blog/2018/02/22/container-terminology-practical-introduction/#h.j2uq93kgxe0e) and [Container Images](https://developers.redhat.com/blog/2018/02/22/container-terminology-practical-introduction/#h.dqlu6589ootw). Podman provides a command line interface (CLI) familiar to anyone who has used the Docker [Container Engine](https://developers.redhat.com/blog/2018/02/22/container-terminology-practical-introduction/#h.6yt1ex5wfo3l). Most users can simply alias Docker to Podman (alias docker=podman) without any problems. Similar to other common [Container Engines](https://developers.redhat.com/blog/2018/02/22/container-terminology-practical-introduction/#h.6yt1ex5wfo3l) (Docker, CRI-O, containerd), Podman relies on an OCI compliant [Container Runtime](https://developers.redhat.com/blog/2018/02/22/container-terminology-practical-introduction/#h.6yt1ex5wfo55) (runc, crun, runv, etc) to interface with the operating system and create the running containers. This makes the running containers created by Podman nearly indistinguishable from those created by any other common container engine.

Containers under the control of Podman can either be run by root or by a non-privileged user. Podman manages the entire container ecosystem which includes pods, containers, container images, and container volumes using the [libpod](https://github.com/containers/podman) library. Podman specializes in all of the commands and functions that help you to maintain and modify OCI container images, such as pulling and tagging. It allows you to create, run, and maintain those containers and container images in a production environment.

## Cross-Platform Support

While the Podman container engine runs natively on Linux, Podman is fully supported on **macOS** and **Windows** through `podman machine`. The `podman machine` command manages a Linux virtual machine (using Apple Hypervisor on macOS, WSL2 or Hyper-V on Windows) where containers actually run. From the user's perspective, the experience is seamless — the Podman CLI on the host communicates transparently with the VM via a REST API.

```
# Initialize and start a Podman machine (macOS/Windows)
podman machine init
podman machine start
```

## Podman Desktop

[Podman Desktop](https://podman-desktop.io/) is a graphical application for managing containers, images, pods, and Kubernetes environments from macOS, Windows, and Linux. It provides a visual interface for common container tasks and can manage Podman machine instances, making it an excellent option for developers who prefer a GUI or are new to containers.

## Key Capabilities

To summarize, Podman makes it easy to find, run, build and share containers.

* **Find**: whether finding a container on docker.io or quay.io, an internal registry server, or directly from a vendor, a couple of [podman search](https://docs.podman.io/en/latest/markdown/podman-search.1.html), and [podman pull](https://docs.podman.io/en/latest/markdown/podman-pull.1.html) commands make it easy
* **Run**: it's easy to consume pre-built images with everything needed to run an entire application, or start from a Linux distribution base image with the [podman run](https://docs.podman.io/en/latest/markdown/podman-run.1.html) command
* **Build**: creating new layers with small tweaks, or major overhauls is easy with [podman build](https://docs.podman.io/en/latest/markdown/podman-build.1.html)
* **Share**: Podman lets you push your newly built containers anywhere you want with a single [podman push](https://docs.podman.io/en/latest/markdown/podman-push.1.html) command
* **Compose**: Podman includes a built-in `podman compose` subcommand that works with standard Docker Compose files, making it easy to manage multi-container applications
