# Simple Dockerfile - Alpine-based

In this section, we are going to use the same Golang application as in the previous section.

## Overview

In the previous section we saw that the size of the Debian-based Docker image was `865MB`. Let's see how we can reduce the final image size, using [Alpine Linux](https://www.alpinelinux.org/).

## Alpine Linux Overview

Alpine Linux is a Linux distribution designed to be small, simple and secure.[3] Unlike most other Linux distributions, Alpine uses [musl](https://en.wikipedia.org/wiki/Musl), [BusyBox](https://en.wikipedia.org/wiki/BusyBox) and [OpenRC](https://en.wikipedia.org/wiki/OpenRC) instead of the more commonly used [glibc](https://en.wikipedia.org/wiki/Glibc), [GNU Core Utilities](https://en.wikipedia.org/wiki/GNU_Core_Utilities) and [systemd](https://en.wikipedia.org/wiki/Systemd) respectively.

## Create a Dockerfile for the application

Below are the differences between the Debian-based and the Alpine-based `Dockerfile`:

```diff
- FROM golang:1.19.3-bullseye
+ FROM golang:1.19.3-alpine3.16
```

The complete Alpine-based `Dockerfile` can be found [here](./Dockerfile.alpine-base).

## Build the image

To build the Alpine-based image:

```bash
$ docker build --tag cms-daq-workshop-alpine \
    -f docs/docker/07-build/dockerfile/Dockerfile.alpine-base \
    app/

[+] Building 1.5s (12/12) FINISHED
 => [internal] load build definition from Dockerfile.alpine-base                                                                                 0.0s
 => => transferring dockerfile: 259B                                                                                                             0.0s
 => [internal] load .dockerignore                                                                                                                0.0s
 => => transferring context: 2B                                                                                                                  0.0s
 => [internal] load metadata for docker.io/library/golang:1.19.3-alpine3.16                                                                      1.4s
 => [internal] load build context                                                                                                                0.0s
 => => transferring context: 81B                                                                                                                 0.0s
 => [1/7] FROM docker.io/library/golang:1.19.3-alpine3.16@sha256:dc4f4756a4fb91b6f496a958e11e00c0621130c8dfbb31ac0737b0229ad6ad9c                0.0s
 => CACHED [2/7] WORKDIR /app                                                                                                                    0.0s
 => CACHED [3/7] COPY go.mod ./                                                                                                                  0.0s
 => CACHED [4/7] COPY go.sum ./                                                                                                                  0.0s
 => CACHED [5/7] RUN go mod download                                                                                                             0.0s
 => CACHED [6/7] COPY *.go ./                                                                                                                    0.0s
 => CACHED [7/7] RUN CGO_ENABLED=0 go build -o /cms-daq-simple-app                                                                               0.0s
 => exporting to image                                                                                                                           0.0s
 => => exporting layers                                                                                                                          0.0s
 => => writing image sha256:80d67b7425d5d8bcf90460010e770f80d5429b3ea94de54933f183e593195f41                                                     0.0s
 => => naming to docker.io/library/cms-daq-workshop-alpine                                                                                       0.0s
```


After the above command finishes successfully, Docker has successfully built our image and assigned a `cms-daq-workshop-alpine` tag to it.

## View Local Images

To list images, run the `docker image ls` command:

```bash
$ docker image ls

REPOSITORY                    TAG       IMAGE ID       CREATED          SIZE
cms-daq-workshop-alpine       latest    80d67b7425d5   2 minutes ago    370MB
cms-daq-workshop-debian       latest    f85afa09b2b4   30 minutes ago   865MB
```

As you can see, the final **Alpine-based Docker image is about x2.3 smaller** than the Debian-based image.

In the next sections we're going to see how we can reduce the final image size even more.

## Run a Container based on the Built Image

After creating the container image, we can instruct Docker to run/start a container, based on the aforementioned image:

```
$ docker run -d --name cms-daq-simple-app-alpine \
    -p 8081:8080 cms-daq-workshop-alpine

19a78dbe7cdeb0285f5316273a3d6a9ce4cb535b0c890d5ac88fe221ce6263a0
```

The above command should create a new container named `cms-daq-simple-app-alpine` (if it doesn't already exist in your system), based on the `cms-daq-workshop-alpine` image on your local machine.

As we're mapping the port `8080` from the container to the port `8081` of our machine, our Golang web application should be reachable from our local machine via:

```
$ curl http://localhost:8081

Hello CMS DAQ Group!
```

Alternatively, [http://localhost:8081/](http://localhost:8081/) should be also accessible from the browser.

## Summary

In this section we went throught:

- a (very) quick overview of [Alpine Linux](https://www.alpinelinux.org/).
- how Alpine-based images are much smaller than Debian-based ones.
