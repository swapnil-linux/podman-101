# Building image using Dockerfile

A Dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image. Using podman build users can create an automated build that executes several command-line instructions in succession. The podman build command builds an image from a Dockerfile.

```
[root@servera htop]# cat Dockerfile 
FROM docker.io/alpine
RUN apk update
RUN apk add htop


[root@servera htop]# podman build .
STEP 1: FROM docker.io/alpine
Getting image source signatures
Copying blob 188c0c94c7c5 done  
Copying config d6e46aa247 done  
Writing manifest to image destination
Storing signatures
STEP 2: RUN apk update
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/community/x86_64/APKINDEX.tar.gz
v3.12.1-82-g0e1cfdcae4 [http://dl-cdn.alpinelinux.org/alpine/v3.12/main]
v3.12.1-83-ge8061d3671 [http://dl-cdn.alpinelinux.org/alpine/v3.12/community]
OK: 12747 distinct packages available
983f5ebc9de4819947493006e3bf4ad1446e29c95ea71362e4a339ce7e9d281e
STEP 3: RUN apk add htop
(1/3) Installing ncurses-terminfo-base (6.2_p20200523-r0)
(2/3) Installing ncurses-libs (6.2_p20200523-r0)
(3/3) Installing htop (2.2.0-r0)
Executing busybox-1.31.1-r19.trigger
OK: 6 MiB in 17 packages
STEP 4: COMMIT
646445d40c3f893a2c6322c2da8e38385e19d25ad9a9eea7ba389678c9d46a5c
646445d40c3f893a2c6322c2da8e38385e19d25ad9a9eea7ba389678c9d46a5c

```

The dot ( . ) at the end represents the current directory where Dockerfile resides, it can also be a path or a URL. Traditionally, the Dockerfile is called Dockerfile and located in the root of the context. You use the -f flag with podman build to point to a Dockerfile anywhere in your file system, URL or with its with a different name in the current directory

```
[root@servera htop]# podman build -f Dockerfile.v1
STEP 1: FROM docker.io/alpine
STEP 2: RUN apk update
--> Using cache 983f5ebc9de4819947493006e3bf4ad1446e29c95ea71362e4a339ce7e9d281e
STEP 3: RUN apk add htop
--> Using cache 646445d40c3f893a2c6322c2da8e38385e19d25ad9a9eea7ba389678c9d46a5c
646445d40c3f893a2c6322c2da8e38385e19d25ad9a9eea7ba389678c9d46a5c

```

You can specify a repository and tag at which to save the new image if the build succeeds:

```
[root@servera htop]# podman build -f Dockerfile.v1 -t myalpine:htop
STEP 1: FROM docker.io/alpine
STEP 2: RUN apk update
--> Using cache 983f5ebc9de4819947493006e3bf4ad1446e29c95ea71362e4a339ce7e9d281e
STEP 3: RUN apk add htop
--> Using cache 646445d40c3f893a2c6322c2da8e38385e19d25ad9a9eea7ba389678c9d46a5c
STEP 4: COMMIT myalpine:htop
646445d40c3f893a2c6322c2da8e38385e19d25ad9a9eea7ba389678c9d46a5c
646445d40c3f893a2c6322c2da8e38385e19d25ad9a9eea7ba389678c9d46a5c

```

podman build runs the instructions in the Dockerfile one-by-one, committing the result of each instruction to a new image if necessary, before finally outputting the ID of your new image. Whenever possible, podman will re-use the intermediate images (cache), to accelerate the docker build process significantly. This is indicated by the **`Using cache`** message in the console output.

## Dockerfile Format

Here is the format of the Dockerfile:

```
# Comment
INSTRUCTION arguments
```

{% hint style="info" %}
The instruction is not case-sensitive. However, the convention is for it to be UPPERCASE to distinguish it from arguments more easily.
{% endhint %}

Podman runs instructions in a Dockerfile in order. A Dockerfile must **start with a FROM instruction**. The FROM instruction specifies the Base Image from which you are building. FROM may only be proceeded by one or more ARG instructions, which declare arguments that are used in FROM lines in the Dockerfile.

Podman treats lines that begin with # as a comment. A # marker anywhere else in a line is treated as an argument.

### `FROM`

The FROM instruction initialises a new build stage and sets the Base Image for subsequent instructions. As such, a valid Dockerfile must start with FROM instruction. The image can be any valid image – it is especially easy to start by pulling an image from the Repositories.

`FROM <image> [AS <name>]`

`FROM <image>[:<tag>] [AS <name>]`

`FROM <image>[@<digest>] [AS <name>]`

* **FROM can appear multiple times** within a single Dockerfile to create multiple images or use one build stage as a dependency for another. Simply make a note of the last image ID output by the commit before each new FROM instruction. Each FROM instruction clears any state created by previous instructions.
* **Optionally a name can be given to a new build stage by adding AS** name to the FROM instruction. The name can be used in subsequent FROM and `COPY --from=<name|index>` instructions to refer to the image built in this stage.

```
# htop on alpine
FROM docker.io/alpine:latest
```

### `RUN`

The RUN instruction will execute any commands in a new layer on top of the current image and commit the results. The resulting committed image will be used for the next step in the Dockerfile. Layering RUN instructions and generating commits conforms to the core concepts of Docker where commits are cheap and containers can be created from any point in an image’s history, much like source control.

```
# htop on alpine
FROM docker.io/alpine:latest
RUN apk update && apk add htop
```

### `CMD`

There can only be one CMD instruction in a Dockerfile. If you list more than one CMD then only the last CMD will take effect. The main purpose of a CMD is to provide defaults for an executing container. These defaults can include an executable, or they can omit the executable, in which case you must specify an ENTRYPOINT instruction as well.

CMD instruction has three forms:

* CMD \["executable","param1","param2"] (exec form, this is the preferred form)
* CMD \["param1","param2"] (as default parameters to ENTRYPOINT)
* CMD command param1 param2 (shell form)

```
# htop on alpine
FROM docker.io/alpine:latest
RUN apk update && apk add htop
CMD ["htop"]
```

### `LABEL`

The LABEL instruction adds metadata to an image. A LABEL is a key-value pair. To include spaces within a LABEL value, use quotes and backslashes as you would in command-line parsing. A few usage examples:

```
LABEL maintainer="swapnil@linux.com"
LABEL organization="Pisces Solutions Private Limited"
LABEL version="1.0"
LABEL description="This image is based on alpine linux pre installed with htop"
```

