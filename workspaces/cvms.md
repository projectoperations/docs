# Docker in workspaces

Standard Coder workspaces run as regular Docker containers. This carries
limitations as to what applications you can run inside your workspace. Most
notably, it's not possible to run Docker securely within regular Docker
containers.

Coder offers an alternative workspace deployment option, called container-based
virtual machines (CVMs), that leverages the
[Sysbox container runtime](https://github.com/nestybox/sysbox). CVMs allow you
to run Docker, Docker Compose, systemd, and other system-level applications
securely within your development containers.

> Coder site managers should review
> [our admin docs](../admin/workspace-management/cvms/index.md) for information
> on enabling Docker in workspaces for your deployment.

## Container-based Virtual Machine (CVM)

By choosing this option, your workspace behaves like a VM or raw host, yet
retains the image, security, and performance properties of typical containers.

To create a workspace capable of securely running system-level applications like
Docker, make sure that the `Run as Container-based Virtual Machine` box is
checked when you create a new workspace. If your admin has enabled CVMs, this
feature will be selected by default whenever you create a new workspace.

![Create CVM](../assets/workspaces/cvm-create.png)

## Disk

Standard workspaces only persist the `/home` directory in your workspace disk.
CVM workspaces have additional levels of persistence:

1. `/var/lib/docker` is stored in your workspace disk and is persisted between
   rebuilds. This prevents shutdowns and rebuilds from purging the Docker cache.

1. The workspace image is itself stored in your workspace disk. Note that this
   data is never directly accessible to you but will still consume data on your
   disk and count towards the size limit.

When setting default disk sizes for [images](../images/index.md), plan for these
additional storage requirements. We recommend treating the workspace as a full
machine, so disk sizes in the range of 50-100 GB are reasonable. This is
especially true if users of the image are storing large Docker caches.
