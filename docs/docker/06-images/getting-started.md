# Docker Images

## Overview

A Docker image is a unit of packaging that contains everything required for an application to run. This includes application code, application dependencies, and OS constructs. If you have an application’s Docker image, the only other thing you need to run that application is a computer running Docker.

## Details

Images are made up of multiple **layers** that are stacked on top of each other and represented as a single object. Inside of the image is a cut-down operating system (OS) and all of the files and dependencies required to run an application. Because containers are intended to be fast and lightweight, images tend to be small.

Images are considered _build-time_ constructs, whereas containers are _run-time_ constructs.

### Images are usually small

**The whole purpose of a container is to run a single application or service**. This means it only needs the code and dependencies of the app/service it is running — it does not need anything else. This results in small images stripped of all non-essential parts.

For example, Docker images do not ship with 6 different shells for you to choose from. In fact, many application images ship without a shell – if the application doesn’t need a shell to run it doesn’t need to be included in the image.

Image also don’t contain a kernel — all containers running on a Docker host share access to the host’s kernel. For these reasons, we sometimes say images contain _just enough operating system_ (usually just OS-related files and filesystem objects).

**The [official Alpine Linux Docker image](https://hub.docker.com/_/alpine) is about 5MB** in size and is an extreme example of how small Docker images can be.

## Listing Docker Images

The `docker image` command is used to list images.

### Examples

#### List All Images

To list all Docker images that are already present in your system:

```bash
docker image ls -a
```

#### List Images by Name and Tag

The `docker images` command takes an optional `[REPOSITORY[:TAG]]` argument that restricts the list to images that match the argument. If you specify `REPOSITORYbut` no `TAG`, the `docker images` command lists all images in the given repository.

For example, to list all images in the `nginx` repository:

```bash
$ docker images nginx:1.23.2

REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
nginx        1.23.2    30d2ef10a9fc   2 weeks ago   135MB
```

#### Filter by Label

The `label` filter matches images based on the presence of a `label` alone or a label and a value.

For example, the following filter matches images with the `com.example.version` label with the `1.0` value:

```bash
docker images --filter "label=com.example.version=1.0"
```

## References

- [`docker image` official docs](https://docs.docker.com/engine/reference/commandline/image/)
