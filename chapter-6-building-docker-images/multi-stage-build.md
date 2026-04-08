# Multi-stage Build

One of the most challenging things about building images is keeping the image size down. Each instruction in the Containerfile adds a layer to the image, and you need to remember to clean up any artifacts you don't need before moving on to the next layer. To write a really efficient Containerfile, you have traditionally needed to employ shell tricks and other logic to keep the layers as small as possible and to ensure that each layer has the artifacts it needs from the previous layer and nothing else.

**THIS IS HOW WE USED TO DO...**

This is a simple go app to print "Hello Podman!".

```
$ cat app.go 
package main

import "fmt"

func main() {
fmt.Println("Hello Podman!")
}
```

And here's our `Containerfile` to build and run this application.

```
$ cat Containerfile 
FROM docker.io/golang:1.23
COPY app.go .
RUN go build -o app app.go
CMD ["./app"]

```

Let's build this

```
$ podman build . -t mygoapp
STEP 1/4: FROM docker.io/golang:1.23
Trying to pull docker.io/library/golang:1.23...
Getting image source signatures
Copying blob 386a066cd84a done   |
Copying blob 75ea84187083 done   |
...
STEP 2/4: COPY app.go .
STEP 3/4: RUN go build -o app app.go
STEP 4/4: CMD ["./app"]
COMMIT mygoapp
--> ab1152b774ee
ab1152b774ee

$ podman run mygoapp
Hello Podman!
```

Check the size of image, it's over 800MB

```
$ podman images
REPOSITORY          TAG       IMAGE ID       CREATED          SIZE
localhost/mygoapp   latest    ab1152b774ee   15 seconds ago   838MB
docker.io/golang    1.23      ef15416724f6   2 weeks ago      836MB

```

**THIS IS HOW WE WOULD LOVE TO DO...**

With multi-stage builds, you use multiple `FROM` statements in your `Containerfile`. Each `FROM` instruction can use a different base, and each of them begins a new stage of the build. You can selectively copy artifacts from one stage to another, leaving behind everything you don't want in the final image. To show how this works, let's adapt the `Containerfile` from the previous section to use multi-stage builds.

```
$ cat Containerfile.multi
FROM docker.io/golang:1.23 AS builder
COPY app.go .
RUN go build -o app app.go

FROM docker.io/alpine:latest
COPY --from=builder /go/app .
CMD ["./app"]
```

You only need the single `Containerfile`. You don't need a separate build script, either. Just run `podman build`.

```
$ podman build . -f Containerfile.multi -t mygoapp:multi
STEP 1/6: FROM docker.io/golang:1.23 AS builder
STEP 2/6: COPY app.go .
--> Using cache
STEP 3/6: RUN go build -o app app.go
--> Using cache
STEP 4/6: FROM docker.io/alpine:latest
STEP 5/6: COPY --from=builder /go/app .
STEP 6/6: CMD ["./app"]
COMMIT mygoapp:multi
--> 40b371535a83
40b371535a83

```

The second `FROM` instruction starts a new build stage with the `alpine:latest` image as its base. The `COPY --from=builder` line copies just the built artifact from the previous stage into this new stage. The Go SDK and any intermediate artifacts are left behind, and not saved in the final image. The end result is a tiny production image.

```
$ podman images
REPOSITORY          TAG       IMAGE ID       CREATED              SIZE
localhost/mygoapp   multi     40b371535a83   About a minute ago   12.1MB
localhost/mygoapp   latest    ab1152b774ee   5 minutes ago        838MB
docker.io/alpine    latest    1d34ffeaf190   4 weeks ago          7.79MB
docker.io/golang    1.23      ef15416724f6   2 weeks ago          836MB

```

Notice the dramatic difference in size — from 838MB down to just 12.1MB!\
