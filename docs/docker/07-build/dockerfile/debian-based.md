# Simple Dockerfile - Debian-based

We'll use a simple Golang web application to demonstrate various `Dockerfile` practices  and techniques.

The application code can be found on the `app/` directory of the Git repo.

## Prerequisites

- You understand basic [Docker concepts](https://docs.docker.com/get-started/overview/).
- You’re familiar with the `Dockerfile` format.
- You have [enabled BuildKit](https://docs.docker.com/build/buildkit/#getting-started) on your machine.
- Clone this Git repository for fetching the Golang code and files needed to build our Docker image.

## Overview

Now that we have a good overview of containers and the Docker platform, let’s take a look at building our first image. An image includes everything you need to run an application – the code or binary, runtime, dependencies, and any other file system objects required.

## Create a Dockerfile for the application

Below is a sample `Dockerfile` (Debian-based) for our Golang web app:

```docker
# Base image for our conteinerized app
FROM golang:1.19.3-bullseye

# Change current directory and use it as the default destination for all subsequent commands.
WORKDIR /app

# Copy files into the image
COPY go.mod ./
COPY go.sum ./

# Execute command to download Go modules
RUN go mod download

# Copy files into the image
COPY *.go ./

# Execute command to build the application binary
RUN go build -o /cms-daq-simple-app

# Inform Docker that the container listens on the specified network port at runtime
EXPOSE 8080

# Command to execute when our image is used to start a container
CMD [ "/cms-daq-simple-app" ]
```

The above `Dockerfile` (without the comments) can be found [here](./Dockerfile.debian-base).

## Build the image

Now that we’ve created our `Dockerfile`, let’s build an image from it. The `docker build` command creates Docker images from the `Dockerfile` and a “context”. A build _context_ is the set of files located in the specified path or URL. The Docker build process can access any of the files located in the context.

The build command optionally takes a `--tag` flag. This flag is used to label the image with a string value, which is easy for humans to read and recognise. If you do not pass a `--tag`, Docker will use `latest` as the default value.


```bash
$ docker build --tag cms-daq-workshop-debian \
    -f docs/docker/07-build/dockerfile/Dockerfile.debian-base \
    app/


[+] Building 0.6s (12/12) FINISHED
 => [internal] load build definition from Dockerfile.debian-base                                                                             0.0s
 => => transferring dockerfile: 244B                                                                                                         0.0s
 => [internal] load .dockerignore                                                                                                            0.0s
 => => transferring context: 2B                                                                                                              0.0s
 => [internal] load metadata for docker.io/library/golang:1.19.3-bullseye                                                                    0.5s
 => [1/7] FROM docker.io/library/golang:1.19.3-bullseye@sha256:34e901ebac66df44ce97b56a9e1bb407307e54fe13e843d6c59da7826ce4dd2c              0.0s
 => [internal] load build context                                                                                                            0.0s
 => => transferring context: 3.51kB                                                                                                          0.0s
 => CACHED [2/7] WORKDIR /app                                                                                                                0.0s
 => CACHED [3/7] COPY go.mod ./                                                                                                              0.0s
 => CACHED [4/7] COPY go.sum ./                                                                                                              0.0s
 => CACHED [5/7] RUN go mod download                                                                                                         0.0s
 => CACHED [6/7] COPY *.go ./                                                                                                                0.0s
 => CACHED [7/7] RUN go build -o /cms-daq-simple-app                                                                                         0.0s
 => exporting to image                                                                                                                       0.0s
 => => exporting layers                                                                                                                      0.0s
 => => writing image sha256:f85afa09b2b4b54afd04c3fd259d1e9213e9dfdd79bd6b245d982bae120aed9b                                                 0.0s
 => => naming to docker.io/library/cms-daq-workshop-debian                                                                                   0.0s
```


Your exact output will vary, but provided there aren’t any errors, you should see the `FINISHED` line in the build output. This means Docker has successfully built our image and assigned a `cms-daq-workshop-debian` tag to it.

## View Local Images

To list images, run the `docker image ls` command:

```bash
$ docker image ls

REPOSITORY                    TAG       IMAGE ID       CREATED          SIZE
cms-daq-workshop-debian       latest    f85afa09b2b4   5 minutes ago   865MB
```

The image that we built in the previous steps, which is a Debian-based image, is 865MB in total.

In a future section we'll see how to reduce the image size (either with Alpine-based images _or_ using multi-stage builds).

## Run a Container based on the Built Image

After creating the container image, we can instruct Docker to run/start a container, based on the aforementioned image:

```
$ docker run -d --name cms-daq-simple-app-debian \
    -p 8080:8080 cms-daq-workshop-debian

7db9bda2aaca424673617c5b3c0b9bdef1001f3f5327e555ddedd8af508d3742
```

The above command should create a new container named `cms-daq-simple-app-debian` (if it doesn't already exist in your system), based on the `cms-daq-workshop-debian` image on your local machine.

As we're mapping the port `8080` from the container to the port `8080` of our machine, our Golang web application should be reachable from our local machine:

```
$ curl http://localhost:8080

Hello CMS DAQ Group!
```

Alternatively, [http://localhost:8080/](http://localhost:8080/) should be also accessible from the browser.

## Summary

As already mentioned, a typical workflow is the following:

```
Dockerfile --> Docker image --> Docker container
```

In this section we saw how to:

- Create a simple Dockerfile for our Golang web app
- Build the Docker image based on our Dockerfile
- Run a Docker container from the image
