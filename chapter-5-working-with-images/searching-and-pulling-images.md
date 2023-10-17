# Searching and pulling images

The Image Registry is a component of the container ecosystem. A registry is a storage and content delivery system, holding named images, available in different tagged versions. For example, the image library/ubuntu, with tags 20.04 and latest. Users interact with a registry by using podman push and pull commands such as, podman pull myregistry.com/stevvooe/batman:voice.

```
[root@servera ~]# podman pull docker.io/ubuntu:20.04
Trying to pull docker.io/ubuntu:20.04...
Getting image source signatures
Copying blob 14428a6d4bcd done  
Copying blob 2c2d948710f2 done  
Copying blob da7391352a9b done  
Copying config f643c72bc2 done  
Writing manifest to image destination
Storing signatures
f643c72bc25212974c16f3348b3a898b1ec1eb13ec1539e10a103e6e217eb2f1


[root@servera ~]# podman pull docker.io/ubuntu:20.10
Trying to pull docker.io/ubuntu:20.10...
Getting image source signatures
Copying blob 5b20aefff2dc done  
Copying blob d8b994c44286 done  
Copying blob b0773f3f1417 done  
Copying config 671495eee4 done  
Writing manifest to image destination
Storing signatures
671495eee4d808a4c78517c3be67355ae0afef2796157cbf42e1efcd0f2a1ccf

[root@servera ~]# podman pull docker.io/ubuntu:latest
Trying to pull docker.io/ubuntu:latest...
Getting image source signatures
Copying blob 2c2d948710f2 skipped: already exists  
Copying blob da7391352a9b skipped: already exists  
Copying blob 14428a6d4bcd [--------------------------------------] 0.0b / 0.0b
Copying config f643c72bc2 done  
Writing manifest to image destination
Storing signatures
f643c72bc25212974c16f3348b3a898b1ec1eb13ec1539e10a103e6e217eb2f1


[root@servera ~]# podman images ubuntu
REPOSITORY                 TAG      IMAGE ID       CREATED       SIZE
docker.io/library/ubuntu   20.10    671495eee4d8   2 weeks ago   82 MB
docker.io/library/ubuntu   latest   f643c72bc252   2 weeks ago   75.3 MB
docker.io/library/ubuntu   20.04    f643c72bc252   2 weeks ago   75.3 MB
[root@servera ~]# 

```

If you notice in the above output, ubuntu:latest and ubuntu:20.10 are two different tags but have the same IMAGE ID. In this case ubuntu:latest is just another tag to the 20.04 image. It does not occupy additional space on the disk.

### Image Search&#x20;

podman search command can be used to search configured registries for images that match the specified TERM. The table of images returned displays the name, truncated description (use --no-trunc to display full description), number of stars awarded, whether the image is official, and whether it is created using automated build.

Multiple registries can be configured for image search and is done in the order it is defined. This configuration is in /etc/containers/registries.conf

```
[root@servera ~]# grep ^registries /etc/containers/registries.conf 
registries = ['registry.access.redhat.com', 'registry.redhat.io', 'docker.io']
```

`podman search` will search for the TERM on all configured registries, use `--limit` option to limit the number of results per registry

```
[root@servera ~]# podman search rhel --limit 3
INDEX        NAME                                           DESCRIPTION                                       STARS   OFFICIAL   AUTOMATED
redhat.com   registry.access.redhat.com/rhel7/rhel-atomic   Red Hat Enterprise Linux Atomic Image is a m...   0                  
redhat.com   registry.access.redhat.com/rhel-atomic         Red Hat Enterprise Linux Atomic Image is a m...   0                  
redhat.com   registry.access.redhat.com/rhel7-minimal       Red Hat Enterprise Linux Minimal Image is a ...   0                  
redhat.io    registry.redhat.io/rhel7/rhel-atomic           Red Hat Enterprise Linux Atomic Image is a m...   0                  
redhat.io    registry.redhat.io/rhel-atomic                 Red Hat Enterprise Linux Atomic Image is a m...   0                  
redhat.io    registry.redhat.io/rhel7-minimal               Red Hat Enterprise Linux Minimal Image is a ...   0                  
docker.io    docker.io/richxsl/rhel7                        RHEL 7 image with minimal installation            29                 
docker.io    docker.io/xplenty/rhel7-pod-infrastructure     registry.access.redhat.com/rhel7/pod-infrast...   1                  
docker.io    docker.io/bluedata/rhel7                       RHEL-7.x base container images                    1           
```

### Image Pull

podman pull command pulls down an image or a repository from a registry. If there is more than one image for a repository (e.g., fedora) then all images for that repository name can be pulled down including any tags using the option `--all-tags`. To download a particular image, or set of images (i.e., a repository), use podman pull. If no tag is provided, it uses the `:latest` tag as a default. This command pulls the debian:latest image:

```
[root@servera ~]# podman pull debian
Trying to pull registry.access.redhat.com/debian...
  name unknown: Repo not found
Trying to pull registry.redhat.io/debian...
  unable to retrieve auth token: invalid username/password: unauthorized: Please login to the Red Hat Registry using your Customer Portal credentials. Further instructions can be found here: https://access.redhat.com/RegistryAuthentication
Trying to pull docker.io/library/debian...
Getting image source signatures
Copying blob 756975cb9c7e done  
Copying config ef05c61d51 done  
Writing manifest to image destination
Storing signatures
ef05c61d51129e3866d5b71b4f44864919dd2b9e5f2644f0a511703182acf8f9
[root@servera ~]# 

```

If no registry is specified in the podman pull command, it tries to pull the image from registry in the order it is configured in the registry.conf.

```
[root@servera ~]# podman pull docker.io/debian
Trying to pull docker.io/debian...
Getting image source signatures
Copying blob 756975cb9c7e done  
Copying config ef05c61d51 done  
Writing manifest to image destination
Storing signatures
ef05c61d51129e3866d5b71b4f44864919dd2b9e5f2644f0a511703182acf8f9
[root@servera ~]# 
```

Images can consist of multiple layers. Layers can be reused by images. For example, the mysql:5.7 image shares few layers with mysql:latest. Pulling the mysql:5.7 image therefore only pulls its metadata and other layers which does not exist locally. The below example show `skipped: already exists` for the shared layer.

```
[root@servera ~]# podman pull docker.io/mysql:5.7
Trying to pull docker.io/mysql:5.7...
Getting image source signatures
Copying blob 938c64119969 skipped: already exists  
Copying blob 29969ddb0ffb skipped: already exists  
Copying blob 852e50cd189d skipped: already exists  
Copying blob 5cdd802543a3 skipped: already exists  
Copying blob a43f41a44c48 skipped: already exists  
Copying blob b79b040de953 skipped: already exists  
Copying blob 7689ec51a0d9 skipped: already exists  
Copying blob aac9d11987ac done  
Copying blob cab9d3fa4c8c done  
Copying blob 36bd6224d58f done  
Copying blob 1b741e1c47de done  
Copying config ae0658fdba done  
Writing manifest to image destination
Storing signatures
ae0658fdbad5fb1c9413c998d8a573eeb5d16713463992005029c591e6400d02

```

### Remove Images

`podman rmi` command removes one or more images from the host node. This does not remove images from a registry. You cannot remove an image of a running container unless you use the -f option, which should not be used.

```
[root@servera ~]# podman rmi docker.io/debian:latest docker.io/debian:stretch
Untagged: docker.io/library/debian:latest
Deleted: ef05c61d51129e3866d5b71b4f44864919dd2b9e5f2644f0a511703182acf8f9
Untagged: docker.io/library/debian:stretch
Deleted: b33ba41eae78980b0cd524b0d80983ac11025f6c5f2776ed4a861c9d5f805ede
[root@servera ~]# 

```
