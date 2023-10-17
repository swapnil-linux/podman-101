# Dockerfile Format Continued...

### `EXPOSE`

`EXPOSE <port> [<port>...]`

The EXPOSE instruction informs podman that the container listens on the specified network ports at runtime. EXPOSE does not make the ports of the container accessible to the host. To do that, you must use either the -p flag to publish a range of ports or the -P flag to publish all of the exposed ports. You can expose one port number and publish it externally under another number.

### `ENV`

`ENV <key> <value>`

`ENV <key>=<value> ...`

The ENV instruction sets the environment variable \<key> to the value \<value>. This value will be in the environment of all “descendant” Dockerfile commands and can be replaced inline in many as well.

The environment variables set using ENV will persist when a container is run from the resulting image. You can view the values using podman inspect, and change them using podman run --env \<key>=\<value>.

### `ADD`

The ADD instruction copies new files, directories or remote file URLs and adds them to the filesystem of the image at the path.&#x20;

ADD obeys the following rules:&#x20;

* If src is a URL and dest does not end with a trailing slash, then a file is downloaded from the URL and copied to dest.
* If src is a URL and dest does end with a trailing slash, then the filename is inferred from the URL and the file is downloaded to dest/filename. For instance, ADD [http://example.com/foobar](http://example.com/foobar) / would create the file /foobar. The URL must have a nontrivial path so that an appropriate filename can be discovered in this case ([http://example.com](http://example.com) will not work).
* If src is a directory, the entire contents of the directory are copied, including filesystem metadata.
* If src is a local tar archive in a recognized compression format (gzip, bzip2 or xz) then it is unpacked as a directory. Resources from remote URLs are not decompressed.
* If src is any other kind of file, it is copied individually along with its metadata. In this case, if dest ends with a trailing slash /, it will be considered a directory and the contents of src will be written at dest/base(src).
* If dest does not end with a trailing slash, it will be considered a regular file and the contents of src will be written at dest.
* If dest doesn’t exist, it is created along with all missing directories in its path.

### `COPY`

Unlike ADD, COPY does a straight-forward, as-is copy of files and folders from the build context into the container. COPY doesn't support URLs as a  \<src> argument so it can't be used to download files from remote locations. Anything that you want to COPY into the container must be present in the local build context. Also, COPY doesn't give any special treatment to archives. If you COPY an archive file it will land in the container exactly as it appears in the build context without any attempt to unpack it.

Optionally COPY accepts a flag —-from=\<name|index> that can be used to set the source location to a previous build stage (created with FROM .. AS \<name>) that will be used instead of a build context sent by the user. The flag also accepts a numeric index assigned for all previous build stages started with FROM instruction.

### `ENTRYPOINT`

Both CMD and ENTRYPOINT instructions define what command gets executed when running a container. There are a few rules that describe their co-operation.

* Dockerfile should specify at least one of CMD or ENTRYPOINT commands.
* CMD should be used as a way of defining default arguments for an ENTRYPOINT command or for executing an ad-hoc command in a container.
* CMD will be overridden when running the container with alternative arguments.

### `VOLUME`

The VOLUME instruction creates a mount point with the specified name and marks it as holding externally mounted volumes from the native host or other containers. The value can be a JSON array, VOLUME \["/var/log/"], or a plain string with multiple arguments, such as VOLUME / var/log or VOLUME /var/log /var/db.

### `USER`

The USER instruction sets the user name (or UID) and optionally the user group (or GID) to use when running the image and for any RUN, CMD and ENTRYPOINT instructions that follow it in the Dockerfile.&#x20;

{% hint style="warning" %}
When the user doesn’t have a primary group then the image (or the next instructions) will be run with the root group.&#x20;
{% endhint %}

### `WORKDIR`

The WORKDIR instruction sets the working directory for any RUN, CMD, ENTRYPOINT, COPY and ADD instructions that follow it in the Dockerfile. If the WORKDIR doesn’t exist, **it will be created** even if it’s not used in any subsequent Dockerfile instruction. The WORKDIR instruction can be used multiple times in a Dockerfile. If a relative path is provided, it will be relative to the path of the previous WORKDIR instruction.

