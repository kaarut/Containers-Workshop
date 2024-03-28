# Data in Docker

By default all files created inside a container are stored on a writable container layer. The container’s writable layer does not persist after the container is deleted, but is suitable for storing ephemeral data that is generated at runtime. This means that:

- The data doesn’t persist when that container no longer exists.
- A container’s writable layer is tightly coupled to the host machine.
- To manage the file system, you need a storage driver that provides a union file system, using the Linux kernel. This extra abstraction reduces performance compared to _data volumes_ which write directly to the filesystem.

Docker has two options for containers to store files on the host machine, so that the files are persisted even after the container stops: _volumes_, and _bind mounts_.

!!!info
    On Linux, _tmpfs mount_ can also be used to store files in the host's system memory.

## Choose the right type of mount

An easy way to visualize the difference among volumes, bind mounts, and `tmpfs` mounts is to think about where the data lives on the Docker host.

![Type of mounts](./overview-types-of-mounts.png)

- **Volumes** are stored in a part of the host filesystem which is _managed by Docker_ (`/var/lib/docker/volumes/` on Linux). Volumes are the best way to persist data in Docker.

- **Bind mounts** may be stored _anywhere_ on the host system. They may even be important system files or directories.

- **`tmpfs`** mounts are stored in the host system’s memory only.

## Container Writable Layer

As we saw on a previous section, the image layers are stacked on top of each other. When you create a new container, you add a new **writable layer** on top of the underlying layers. This layer is often called the "container layer". All changes made to the running container, such as writing new files, modifying existing files, and deleting files, are written to this thin writable container layer. The diagram below shows a container based on an `ubuntu:15.04` image.

![Container Writable Layer](./container-writable-layer.jpeg)

The **major difference between a container and an image** is the top writable layer. All writes to the container that add new or modify existing data are stored in this writable layer. When the container is deleted, the writable layer is also deleted. The underlying image remains unchanged.
