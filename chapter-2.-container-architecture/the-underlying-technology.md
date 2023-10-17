# The Underlying Technology

There’s an old saying that “nobody runs an operating system just to run an operating system” and the same is true with containers. It’s the workload running on top of an operating system or in a container that’s interesting and valuable.

## Linux Kernel Namespace

**Namespaces** are a feature of the Linux kernel that partitions kernel resources such that one set of processes sees one set of resources while another set of processes sees a different set of resources. Namespaces provides the isolation containerised applications. When you run a container, Podman creates a set of namespaces for that container. These namespaces provide a layer of isolation. Each aspect of a container runs in a separate namespace and its access is limited to that namespace. These namespaces are:

#### 1. Mount (mnt)

Mount namespaces control mount points. Upon creation the mounts from the current mount namespace are copied to the new namespace, but mount points created afterwards do not propagate between namespaces (using shared subtrees, it is possible to propagate mount points between namespaces).

#### 2. Process ID (pid)

The PID namespace provides processes with an independent set of process IDs (PIDs) from other namespaces. PID namespaces are nested, meaning when a new process is created it will have a PID for each namespace from its current namespace up to the initial PID namespace. Hence the initial PID namespace is able to see all processes, albeit with different PIDs than other namespaces will see processes with.

The first process created in a PID namespace is assigned the process id number 1 and receives most of the same special treatment as the normal init process, most notably that orphaned processes within the namespace are attached to it. This also means that the termination of this PID 1 process will immediately terminate all processes in its PID namespace and any descendants.

#### 3. Network (net)

Network namespaces virtualize the network stack. Each network namespace will have a private set of IP addresses, its own routing table, socket listing, connection tracking table, firewall, and other network-related resources.

#### 4. Interprocess Communication (ipc)

IPC namespaces isolate processes from SysV style inter-process communication. This prevents processes in different IPC namespaces from using, for example, the SHM family of functions to establish a range of shared memory between the two processes. Instead each process will be able to use the same identifiers for a shared memory region and produce two such distinct regions.

#### 5. UTS

UTS (UNIX Time-Sharing) namespaces allow a single system to appear to have different host and domain names to different processes.

#### 6. User ID (user)

User namespaces are a feature to provide both privilege isolation and user identification segregation across multiple sets of processes available since kernel 3.8. With administrative assistance it is possible to build a container with seeming administrative rights without actually giving elevated privileges to user processes. Like the PID namespace, user namespaces are nested and each new user namespace is considered to be a child of the user namespace that created it.

A user namespace contains a mapping table converting user IDs from the container's point of view to the system's point of view. This allows, for example, the root user to have user id 0 in the container but is actually treated as user id 14678 by the system for ownership checks. A similar table is used for group id mappings and ownership checks.

## Control Groups

OCI Containers on Linux also relies on another technology called control groups (cgroups). A cgroup limits an application to a specific set of resources. Control groups allow to share available hardware resources to containers and optionally enforce limits and constraints. For example, you can limit the memory available to a specific container.

## Union FileSystem

Union file systems, or UnionFS, are file systems that operate by creating layers, making them very lightweight and fast. Docker Engine uses UnionFS to provide the building blocks for containers. Docker Engine can use multiple UnionFS variants, including AUFS, btrfs, vfs, and DeviceMapper.

## SELinux

On SELinux enabled systems like Red Hat Enterprise Linux and CentOS SELinux controls access to processes by Type and Level.&#x20;

**SELinux labels consist of 4 parts:**

`User:Role:Type:level`

Type enforcement is a kind of enforcement in which rules are based on process type. It works in the following way. The default type for a confined container process is svirt\_lxc\_net\_t. This type is permitted to read and execute all files types under /usr and most types under/etc. svirt\_lxc\_net\_t is permitted to use the network but is not permitted to read content under/var, /home, /root, /mnt … svirt\_lxc\_net\_t is permitted to write only to files labeled `container_file_t` . All files in a container are labeled by default as `container_file_t`.&#x20;

Multi-Category Security (MCS) Separation is sometimes called svirt. It works in the following way. A unique value is assigned to the level field of the SELinux label of each container. By default each container is assigned the MCS Level equivalent to the PID of the docker process that starts the container.

The standard targeted policy includes rules that dictate that the MCS Labels of the process must dominate the MCS label of the target. The target is usually a file. The MCS Label usually looks something like `s0:c1,c2` Such a label would Dominate files labeled `s0, s0:c1, s0:c2, s0:c1,c2`. It would not, however, dominate `s0:c1,c3`. All MCS Labels are required to use two Categories. This guarantees that no two containers can have the same MCS Label by default.

