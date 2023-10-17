# Multi-stage Build

One of the most challenging things about building images is keeping the image size down. Each instruction in the Dockerfile adds a layer to the image, and you need to remember to clean up any artifacts you don’t need before moving on to the next layer. To write a really efficient Dockerfile, you have traditionally needed to employ shell tricks and other logic to keep the layers as small as possible and to ensure that each layer has the artifacts it needs from the previous layer and nothing else.

**THIS IS HOW WE USED TO DO...**

This is a simple go app to print "Hello Docker!".

```
[root@node1 multistage]# cat app.go 
package main

import "fmt"

func main() {
fmt.Println("Hello Docker!")
}
```

And heres our `Dockerfile` to build and run this application.

```
[root@node1 multistage]# cat Dockerfile 
FROM golang:1.7.3
COPY app.go .
RUN go build -o app app.go
CMD ["./app"]
root@ubuntu:~/multistage#

```

Lets build this

```
[root@node1 multistage]# podman build . -t mygoapp
1.7.3: Pulling from library/golang
386a066cd84a: Pull complete
75ea84187083: Pull complete
88b459c9f665: Pull complete
a31e17eb9485: Pull complete
457559cc1d69: Pull complete
47fe51a74a06: Pull complete
08dacccac43c: Pull complete
Digest: sha256:340212e9c5d062f3bfe58ff02768da70234ea734bd022a357ee6be2a6d963505Status: Downloaded newer image for golang:1.7.3
---> ef15416724f6Step 2/4 : COPY app.go .
---> c50e64d59b04
Removing intermediate container 8198404914c9
Step 3/4 : RUN go build -o app app.go
---> Running in 685898ecf74f
---> 739f7b2a47c7
Removing intermediate container 685898ecf74f
Step 4/4 : CMD ./app
---> Running in 66334e6ca5d8
---> ab1152b774ee
Removing intermediate container 66334e6ca5d8
Successfully built ab1152b774ee
Successfully tagged mygoapp:latest
[root@node1 multistage]#

[root@node1 multistage]# podman run mygoapp
Hello Docker!
[root@node1 multistage]#
```

Check the size of image, its 674MB

```
[root@node1 multistage]# podman image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mygoapp             latest              efdc2b679aa6        15 seconds ago      674MB
golang              1.7.3               ef15416724f6        8 months ago        672MB
[root@node1 multistage]#
```

**THIS IS HOW WE WOULD LOVE TO DO...**

With multi-stage builds, you use multiple `FROM` statements in your `Dockerfile`. Each `FROM` instruction can use a different base, and each of them begins a new stage of the build. You can selectively copy artifacts from one stage to another, leaving behind everything you don’t want in the final image. To show how this works, Let’s adapt the `Dockerfile` from the previous section to use multi-stage builds.

```
[root@node1 multistage]# cat Dockerfile.multi
FROM golang:1.7.3 AS builder
COPY app.go .
RUN go build -o app app.go

FROM alpine:latest
COPY --from=builder /go/app .
CMD ["./app"]
```

You only need the single `Dockerfile`. You don’t need a separate build script, either. Just run `docker build`.

```
[root@node1 multistage]# podman build . -f Dockerfile.multi -t mygoapp:multi
Step 1/6 : FROM golang:1.7.3 AS builder
 ---> ef15416724f6
Step 2/6 : COPY app.go .
 ---> Using cache
 ---> 15f89542f308
Step 3/6 : RUN go build -o app app.go
 ---> Using cache
 ---> 09d91ef7f1a1
Step 4/6 : FROM alpine:latest
latest: Pulling from library/alpine
88286f41530e: Pull complete
Digest: sha256:1072e499f3f655a032e88542330cf75b02e7bdf673278f701d7ba61629ee3ebe
Status: Downloaded newer image for alpine:latest
 ---> 7328f6f8b418
Step 5/6 : COPY --from=builder /go/app .
 ---> adee51a03a34
Removing intermediate container de57adb9658f
Step 6/6 : CMD ./app
 ---> Running in 42c17df73436
 ---> 40b371535a83
Removing intermediate container 42c17df73436
Successfully built 40b371535a83
Successfully tagged mygoapp:multi
[root@node1 multistage]#
```

The second `FROM` instruction starts a new build stage with the `alpine:latest` image as its base. The `COPY --from=builder` line copies just the built artifact from the previous stage into this new stage. The Go SDK and any intermediate artifacts are left behind, and not saved in the final image. The end result is a tiny production image.

```
[root@node1 multistage]# podman images
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
mygoapp             multi               40b371535a83        About a minute ago   5.6MB
mygoapp             latest              efdc2b679aa6        5 minutes ago        674MB
alpine              latest              7328f6f8b418        3 weeks ago          3.97MB
golang              1.7.3               ef15416724f6        8 months ago         672MB
[root@node1 multistage]#

```

Notice the difference in size. \




