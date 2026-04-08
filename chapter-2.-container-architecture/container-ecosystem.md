# Container Ecosystem

## Container

**A container is a runnable instance of an image**. You can create, run, stop, move, or delete a container using Podman. You can connect a container to one or more networks, attach storage to it, or even create a new image based on its current state.

By default, a container is relatively well isolated from other containers and its host machine. You can control how isolated a container's network, storage, or other underlying subsystems are from other containers or from the host machine.

**A container is defined by its image** as well as any configuration options you provide to it when you create or run it. When a container stops, any changes to its state that are not stored in persistent storage disappear.

## Image

**An image is a read-only template with instructions for creating an OCI container.** Often, an image is based on another image, with some additional customization. For example, you may build an image which is based on the UBI (Universal Base Image) image, but installs the Apache web server and your application, as well as the configuration details needed to make your application run.

**You might create your own images or you might only use those created by others and published in a registry.** To build your own image, you create a Containerfile (also known as Dockerfile) with a simple syntax for defining the steps needed to create the image and run it. **Each instruction in a Containerfile creates a layer in the image**. When you change the Containerfile and rebuild the image, only those layers which have changed are rebuilt. This is part of what makes images so lightweight, small, and fast, when compared to other virtualization technologies.

## Registry

A container image registry stores images. Docker Hub (docker.io), quay.io, registry.access.redhat.com, and ghcr.io (GitHub Container Registry) are public registries that anyone can use. You can even run your own private registry.

Podman's registry configuration is defined in `/etc/containers/registries.conf` using TOML format:

```toml
# /etc/containers/registries.conf

# Registries to search when pulling unqualified images (e.g., "podman pull alpine")
unqualified-search-registries = ["registry.access.redhat.com", "registry.redhat.io", "docker.io"]

# Example: configuring a specific registry
[[registry]]
location = "registry.access.redhat.com"
prefix = "registry.access.redhat.com"

# Example: configuring an insecure (HTTP) private registry
[[registry]]
location = "my-private-registry.example.com:5000"
insecure = true
```

When you use the podman pull or podman run commands, the required images are pulled from your configured registries. When you use the podman push command, your image is pushed to the specified registry.
