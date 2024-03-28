# Dockerfile Best Practices

!!! note
    These are some of the available Dockerfile best practices. The list might be updated in the future with more items.

## Best Practices
### Dockerignore

Sometimes you don't want to include all files present in the local source directory in the build context. You can use the `.dockerignore` file for this. This is used to exclude files and directories from the build context.

This file supports exclusion patterns similar to `.gitignore` files. For information on creating one, see the [`.dockerignore` file documentation](https://docs.docker.com/engine/reference/builder/#dockerignore-file).

### Use Multi-stage Builds

[Multi-stage builds](./multi-stage-builds.md) allow you to drastically reduce the size of your final image, without struggling to reduce the number of intermediate layers and files.

### Don’t install unnecessary packages

To reduce complexity, dependencies, file sizes, and build times, avoid installing extra or unnecessary packages just because they might be “nice to have”. For example, you don’t need to include a text editor in a database image.

### Minimize the number of layers

Only the instructions `RUN`, `COPY`, `ADD` create layers. Other instructions create temporary intermediate images, and do not increase the size of the build.

Where possible, use [multi-stage builds](./multi-stage-builds.md), and only copy the artifacts you need into the final image. This allows you to include tools and debug information in your intermediate build stages without increasing the size of the final image.

### User

If a service can run without privileges, use `USER` to change to a non-root user. Start by creating the user and group in the `Dockerfile` with something like:

```
RUN groupadd -r postgres && useradd --no-log-init -r -g postgres postgres
```

!!! tip
    Consider an explicit UID/GID

    Users and groups in an image are assigned a non-deterministic UID/GID in that the “next” UID/GID is assigned regardless of image rebuilds. So, if it’s critical, you should assign an explicit UID/GID.

### ADD or COPY

Although `ADD` and `COPY` are functionally similar, generally speaking, `COPY` is preferred. That’s because it’s more transparent than `ADD`. `COPY` only supports the basic copying of local files into the container, while `ADD` has some features (like local-only tar extraction and remote URL support) that are not immediately obvious. Consequently, the best use for `ADD` is local tar file auto-extraction into the image, as in `ADD rootfs.tar.xz /`.

### Create your own base image

If you have multiple images with a lot in common, consider creating your own [base image](https://docs.docker.com/develop/develop-images/baseimages/) with the shared components, and basing your unique images on that. Docker only needs to load the common layers once, and they are cached. This means that your derivative images use memory on the Docker host more efficiently and load more quickly.

## Resources

- [Best practices for writing Dockerfiles
 (Official Docker docs)](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
