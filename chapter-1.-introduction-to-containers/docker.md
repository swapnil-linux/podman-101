# Docker

## Docker

DotCloud — now **Docker, Inc.** — released Docker as an attempt to leverage Linux namespacing to provide software engineers with a standard box. Linux namespaces are a feature of the Linux kernel that allows for one set of programs to see one set of computing resources, and another set of programs to see another set of computing resources. This feature was present in the kernel long before Docker was released, but using raw kernel capabilities directly is not high on the list of priorities of most developers. However, **Docker provided an interface that made almost no mention of Linux namespaces and aimed to be as easy to use as possible**.

Docker open-sourced three things that allowed their **container tools to be widely adopted**:

* A standard container format; (Docker Image)
* Tools for developers to build containers; (Dockerfile / Docker Build)
* Tools for operators to run containers; (Docker CLI/API)

Docker's standard box for packaging software is a file archive inside of which developers can put all that their program needs — code, libraries, or any other file — as well as basic instructions for starting the application. **This archive is called a container image**. Once built, **images are static**: they never change. This immutability is essential because it allows developers to version their images, and operators not to worry about an image changing when the application it contains is running. Docker's image format uses a copy-on-write filesystem to make images immutable, but this is an implementation detail and goes beyond the scope of this article.

**To build container images with Docker**, developers need to write a recipe called a **Dockerfile**. This recipe is a simple text file stored alongside a software component's code that provides instructions on how to package it in a standalone manner. Only data mentioned in the Dockerfile ends up in the final image, so developers have complete visibility over what the image contains. The steps described inside the Dockerfile should be as explicit as possible about dependencies — like the choice of Linux distribution, or library versions — as to make the build process as deterministic and repeatable as possible. Developers can collaborate across heterogeneous working environments and be confident that they always package their code in the same way. This confidence improves productivity: automated pipelines reliably build container images just as well as developer laptops.



After building an image, developers can share it over the network, and operators can copy it to the machine where they want to run the packaged software. There, they can **use Docker to open up the image and run the application that is inside**, without needing to install anything onto the underlying system other than Docker itself. **A container runtime is a program that takes a container image and runs the application found inside**. Docker's container runtime uses Linux namespaces to create separate environments for each running container.

A program running inside a container sees no other program running on the machine. In reality, all containers share the underlying computer and operating system. Still, the Linux kernel makes sure that applications running in separate containers know nothing about one another: **they have different filesystems, network and storage interfaces, processors, and even memory**. This separation is purely logical, so **it costs very little in performance**. The files inside a container image serve as a read-only filesystem for containers created from that image. Programs running inside a container cannot see any other files. Code running inside a container behaves independently of the underlying system: **the application is entirely separate from the infrastructure**.

Remember the separation of concerns mentioned earlier. **Docker allows developers to put anything their application might need inside a container**, without needing to know how or where the container might run. Conversely, operators can deploy a container based on any image, regardless of its contents or how exactly developers built it. This separation of concerns has paved the way for massive amounts of automation: developers need only provide their code and a Dockerfile so that their software can be packaged, deployed, and run reliably by automated pipelines.
