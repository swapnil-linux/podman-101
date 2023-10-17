# Virtualization vs Containerization

## Virtualization

Virtualization began in the 1960s, as a method of logically dividing the system resources provided by mainframe computers between different applications. Since then, the meaning of the term has broadened. **Traditional Virtualization provides us with a Virtual Hardware where we select and Install Operating System.**

![](<../.gitbook/assets/Screen Shot 2020-12-08 at 7.44.54 pm.png>)

Virtual machines (VMs) are an abstraction of physical hardware, turning one server into many servers. The hypervisor allows multiple VMs to run on a single machine. **Each VM includes a full copy of an operating system,** one or more apps, necessary binaries and libraries - taking up tens of GBs. VMs can also be slow to boot.

## Containerization

Containers differ from Virtualization in the way they are implemented. Containerisation (also known as Operating-system-level virtualization) is a computer virtualization method in which the kernel of an operating system allows the existence of multiple isolated user-space instances, instead of just one. Such instances, which are called containers, virtualization engines (VEs) or jails (FreeBSD jail or chroot jail), may look like real computers from the point of view of programs running in them. **Containers provide a virtual operating system on the same hardware sharing the same base operating system kernel.**

Container implementation was first available in 1982 as chroot in most Unix like operating systems, in 2004 as zones in Solaris and became more popular after implementation as Docker containers since 2013.\


| **Mechanism**                   | **Operating System**             | **Available Since** |
| ------------------------------- | -------------------------------- | ------------------- |
| chroot                          | most UNIX-like operating systems | 1982                |
| jail                            | FreeBSD, DragonFly BSD           | 2000                |
| Zones                           | OpenSolaris, Solaris             | 2004                |
| OpenVZ                          | Linux                            | 2005                |
| WPARs                           | AIX                              | 2007                |
| LXC                             | Linux                            | 2008                |
| Docker                          | Linux                            | 2013                |
| Open Container Initiative (OCI) | Linux                            | 2015/2017           |

Containers and virtual machines have similar resource isolation and allocation benefits, but function differently because containers virtualize the operating system instead of hardware, containers are more portable and efficient and lightweight.

![](<../.gitbook/assets/Screen Shot 2020-12-08 at 8.07.40 pm.png>)

Containers are an abstraction at the app layer that packages code and dependencies together. Multiple containers can run on the same machine and share the OS kernel with other containers, **each running as isolated processes in userspace**. Containers take up less space than VMs (container images are typically tens of MBs in size), and start almost instantly.

