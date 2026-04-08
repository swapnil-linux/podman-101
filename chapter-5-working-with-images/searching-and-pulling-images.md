# Searching and pulling images

The Image Registry is a component of the container ecosystem. A registry is a storage and content delivery system, holding named images, available in different tagged versions. For example, the image library/ubuntu, with tags 24.04 and latest. Users interact with a registry by using podman push and pull commands such as, podman pull myregistry.com/stevvooe/batman:voice.

```
$ podman pull docker.io/ubuntu:24.04
Trying to pull docker.io/library/ubuntu:24.04...
Getting image source signatures
Copying blob 857cc8cb19c0 done   |
Copying config 59ab366372 done   |
Writing manifest to image destination
59ab366372d56772eb54e426183435e6b0642152cb449ec7ab52473af8ca6e3f

$ podman pull docker.io/ubuntu:22.04
Trying to pull docker.io/library/ubuntu:22.04...
Getting image source signatures
Copying blob 5b20aefff2dc done   |
Copying config a24be041e5 done   |
Writing manifest to image destination
a24be041e5e10596d8ede1ca6539bc8ef0a3e8e5e3bf8f2e12a2ab41e8a7a0ca

$ podman pull docker.io/ubuntu:latest
Trying to pull docker.io/library/ubuntu:latest...
Getting image source signatures
Copying blob 857cc8cb19c0 skipped: already exists  
Copying config 59ab366372 done   |
Writing manifest to image destination
59ab366372d56772eb54e426183435e6b0642152cb449ec7ab52473af8ca6e3f


$ podman images ubuntu
REPOSITORY                 TAG      IMAGE ID       CREATED       SIZE
docker.io/library/ubuntu   22.04    a24be041e5e1   2 weeks ago   77.9 MB
docker.io/library/ubuntu   latest   59ab366372d5   2 weeks ago   78.1 MB
docker.io/library/ubuntu   24.04    59ab366372d5   2 weeks ago   78.1 MB

```

If you notice in the above output, ubuntu:latest and ubuntu:24.04 are two different tags but have the same IMAGE ID. In this case ubuntu:latest is just another tag to the 24.04 image. It does not occupy additional space on the disk.

### Image Search&#x20;

podman search command can be used to search configured registries for images that match the specified TERM. The table of images returned displays the name, truncated description (use --no-trunc to display full description), number of stars awarded, whether the image is official, and whether it is created using automated build.

Multiple registries can be configured for image search. The list of registries to search is defined in `/etc/containers/registries.conf` using the `unqualified-search-registries` setting:

```toml
# /etc/containers/registries.conf
unqualified-search-registries = ["registry.access.redhat.com", "registry.redhat.io", "docker.io"]
```

`podman search` will search for the TERM on all configured registries, use `--limit` option to limit the number of results per registry

```
$ podman search rhel --limit 3
INDEX        NAME                                           DESCRIPTION                                       STARS   OFFICIAL   AUTOMATED
redhat.com   registry.access.redhat.com/ubi9/ubi            The Universal Base Image is designed and MDN...   0                  
redhat.com   registry.access.redhat.com/ubi9/ubi-minimal    The Universal Base Image Minimal is a stripp...   0                  
redhat.com   registry.access.redhat.com/ubi9/ubi-init       The Universal Base Image Init is designed to...   0                  
redhat.io    registry.redhat.io/ubi9/ubi                    The Universal Base Image is designed and MDN...   0                  
redhat.io    registry.redhat.io/ubi9/ubi-minimal            The Universal Base Image Minimal is a stripp...   0                  
redhat.io    registry.redhat.io/ubi9/ubi-init               The Universal Base Image Init is designed to...   0                  
docker.io    docker.io/richxsl/rhel7                        RHEL 7 image with minimal installation            29                 
docker.io    docker.io/xplenty/rhel7-pod-infrastructure     registry.access.redhat.com/rhel7/pod-infrast...   1                  
docker.io    docker.io/bluedata/rhel7                       RHEL-7.x base container images                    1           
```

### Image Pull

podman pull command pulls down an image or a repository from a registry. If there is more than one image for a repository (e.g., fedora) then all images for that repository name can be pulled down including any tags using the option `--all-tags`. To download a particular image, or set of images (i.e., a repository), use podman pull. If no tag is provided, it uses the `:latest` tag as a default. This command pulls the debian:latest image:

```
$ podman pull docker.io/debian
Trying to pull docker.io/library/debian:latest...
Getting image source signatures
Copying blob 756975cb9c7e done   |
Copying config ef05c61d51 done   |
Writing manifest to image destination
ef05c61d51129e3866d5b71b4f44864919dd2b9e5f2644f0a511703182acf8f9

```

{% hint style="info" %}
It is best practice to always use fully-qualified image names (e.g., `docker.io/library/debian`) to avoid ambiguity. When pulling without a registry prefix, Podman searches registries listed in `unqualified-search-registries` in order, which may be slow or fail on registries that don't carry the image.
{% endhint %}

Images can consist of multiple layers. Layers can be reused by images. For example, the mysql:8.0 image shares layers with mysql:latest. Pulling the mysql:8.0 image therefore only pulls its metadata and other layers which do not exist locally. The below example shows `skipped: already exists` for the shared layers.

```
$ podman pull docker.io/mysql:8.0
Trying to pull docker.io/library/mysql:8.0...
Getting image source signatures
Copying blob 938c64119969 skipped: already exists  
Copying blob 29969ddb0ffb skipped: already exists  
Copying blob 852e50cd189d skipped: already exists  
Copying blob 5cdd802543a3 skipped: already exists  
Copying blob a43f41a44c48 skipped: already exists  
Copying blob b79b040de953 skipped: already exists  
Copying blob 7689ec51a0d9 skipped: already exists  
Copying blob aac9d11987ac done   |
Copying blob cab9d3fa4c8c done   |
Copying blob 36bd6224d58f done   |
Copying config ae0658fdba done   |
Writing manifest to image destination
ae0658fdbad5fb1c9413c998d8a573eeb5d16713463992005029c591e6400d02

```

### Remove Images

`podman rmi` command removes one or more images from the host node. This does not remove images from a registry. You cannot remove an image of a running container unless you use the -f option, which should not be used.

```
$ podman rmi docker.io/debian:latest
Untagged: docker.io/library/debian:latest
Deleted: ef05c61d51129e3866d5b71b4f44864919dd2b9e5f2644f0a511703182acf8f9

```
