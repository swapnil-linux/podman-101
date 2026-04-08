# Practice: Installing and testing Podman

## Fedora

Podman is included by default on Fedora. If not already installed:

```
sudo dnf -y install podman
```

## RHEL 9 / CentOS Stream 9

Install the container-tools module:

```
sudo dnf install -y container-tools
```

## Ubuntu / Debian

Podman is available in the official repositories for Ubuntu 22.04 and newer, and Debian 11+.

```
sudo apt-get -y update
sudo apt-get -y install podman
```

## macOS

On macOS, install Podman via [Homebrew](https://brew.sh/) or download [Podman Desktop](https://podman-desktop.io/):

```bash
# Install via Homebrew
brew install podman

# Initialize and start the Podman machine (Linux VM)
podman machine init
podman machine start
```

Alternatively, download and install [Podman Desktop](https://podman-desktop.io/) which provides a graphical interface and manages the Podman machine for you.

## Windows

On Windows, install Podman using the official MSI installer from the [Podman releases page](https://github.com/containers/podman/releases) or use [Podman Desktop](https://podman-desktop.io/).

```powershell
# After installing, initialize and start the Podman machine (uses WSL2 or Hyper-V)
podman machine init
podman machine start
```

## Testing Podman

1. Check podman version

```
$ podman version
Client:       Podman Engine
Version:      5.4.2
API Version:  5.4.2
Go Version:   go1.23.6
Built:        Fri Feb 21 21:07:04 2025
OS/Arch:      linux/amd64
```

2\. Pull alpine image using podman pull command

```
$ podman pull docker.io/alpine
Trying to pull docker.io/library/alpine:latest...
Getting image source signatures
Copying blob c6a83fedfae6 done   |
Copying config 1d34ffeaf1 done   |
Writing manifest to image destination
1d34ffeaf190be23d3de5a8de0a85e3b15a3da1b9b7b21ab5fe8717ce0e3b12e
```

3\. Create a test container using podman run command

```
$ podman run --name test-container-1 docker.io/alpine echo Hello World
Hello World
```

4\. List the container using podman ps command

```
$ podman ps -a
CONTAINER ID  IMAGE                            COMMAND          CREATED         STATUS                    PORTS  NAMES
a1b2c3d4e5f6  docker.io/library/alpine:latest  echo Hello Worl  10 seconds ago  Exited (0) 9 seconds ago         test-container-1
```

5\. Remove the container and image

```
$ podman rm test-container-1
a1b2c3d4e5f6
```

```
$ podman rmi docker.io/alpine
Untagged: docker.io/library/alpine:latest
Deleted: 1d34ffeaf190be23d3de5a8de0a85e3b15a3da1b9b7b21ab5fe8717ce0e3b12e
```

_**Congratulations! you have successfully installed and tested podman.**_
