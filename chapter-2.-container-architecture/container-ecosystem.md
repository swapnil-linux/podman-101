# Container Ecosystem

## Container

**A container is a runnable instance of an image**. You can create, run, stop, move, or delete a container using the Podman. You can connect a container to one or more networks, attach storage to it, or even create a new image based on its current state.

By default, a container is relatively well isolated from other containers and its host machine. You can control how isolated a container’s network, storage, or other underlying subsystems are from other containers or from the host machine.

**A container is defined by its image** as well as any configuration options you provide to it when you create or run it. When a container stops, any changes to its state that are not stored in persistent storage disappear.

## Image

**An image is a read-only template with instructions for creating an OCI container.** Often, an image is based on another image, with some additional customization. For example, you may build an image which is based on the CentOS image, but installs the Apache web server and your application, as well as the configuration details needed to make your application run.

**You might create your own images or you might only use those created by others and published in a registry.** To build your own image, you create a Dockerfile with a simple syntax for defining the steps needed to create the image and run it. **Each instruction in a Dockerfile creates a layer in the image**. When you change the Dockerfile and rebuild the image, only those layers which have changed are rebuilt. This is part of what makes images so lightweight, small, and fast, when compared to other virtualization technologies.

## Registry

A container image registry stores images. Docker Hub, quay.io, registry.access.redhat.com and gcr.io are public registries that anyone can use. Podman is configured to look for images on registry.access.redhat.com,  registry.redhat.io and docker.io by default. You can even run your own private registry.

```
[root@servera ~]# grep ^registries /etc/containers/registries.conf 
registries = ['registry.access.redhat.com', 'registry.redhat.io', 'docker.io']
```

When you use the podman pull or podman run commands, the required images are pulled from your configured registry. When you use the podman push command, your image is pushed to your configured registry.

