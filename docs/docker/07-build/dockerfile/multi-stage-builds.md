# Multi-stage Builds

In this section, we are going to use the same Golang application as in the previous section.

## Overview

In the previous section we saw that the size of the Debian-based Docker image was `865MB` and the Alpine-based was `370MB` in total. Let's see how we can reduce the final image size, using Docker's [multi-stage builds](https://docs.docker.com/build/building/multi-stage/) and the [`scratch` base image](https://hub.docker.com/_/scratch), since Golang produces a single binary that contains all the necessary code to run our web application.

One of the most challenging things about building images is keeping the image size down. Each `RUN`, `COPY`, and `ADD` instruction in the Dockerfile adds a layer to the image

It was actually very common to have one Dockerfile to use for development (which contained everything needed to build your application), and a slimmed-down one to use for production, which only contained your application and exactly what was needed to run it. This has been referred to as the _“builder pattern”_. Maintaining two Dockerfiles is not ideal.

## Create a Dockerfile for the application

Below is a (sample) multi-stage `Dockerfile` for our Golang app:

```docker
#######################
##### Build Image #####
#######################

# Use the Alpine-based image as our build base
FROM golang:1.19.3-alpine3.16 AS build

WORKDIR /app

COPY go.mod ./
COPY go.sum ./
RUN go mod download

COPY *.go ./

RUN CGO_ENABLED=0 go build -o /cms-daq-simple-app


########################
### Production Image ###
########################

# Use the `scratch` base for our final production image.
FROM scratch
WORKDIR /

# Copy file from the "build" stage to the new final image.
COPY --from=build /cms-daq-simple-app /cms-daq-simple-app

EXPOSE 8080
ENTRYPOINT ["/cms-daq-simple-app"]
```

The complete multi-stage `Dockerfile` can be found [here](./Dockerfile.multistage).

## Build the image

To build the multi-stage Docker image:

```bash
$ docker build --tag cms-daq-workshop-multi-stage \
    -f docs/docker/07-build/dockerfile/Dockerfile.multistage \
    app/


[+] Building 1.1s (13/13) FINISHED
 => [internal] load build definition from Dockerfile.multistage                                                                                                                                                  0.0s
 => => transferring dockerfile: 48B                                                                                                                                                                              0.0s
 => [internal] load .dockerignore                                                                                                                                                                                0.0s
 => => transferring context: 2B                                                                                                                                                                                  0.0s
 => [internal] load metadata for docker.io/library/golang:1.19.3-alpine3.16                                                                                                                                      1.0s
 => [internal] load build context                                                                                                                                                                                0.0s
 => => transferring context: 81B                                                                                                                                                                                 0.0s
 => [build 1/7] FROM docker.io/library/golang:1.19.3-alpine3.16@sha256:dc4f4756a4fb91b6f496a958e11e00c0621130c8dfbb31ac0737b0229ad6ad9c                                                                          0.0s
 => CACHED [build 2/7] WORKDIR /app                                                                                                                                                                              0.0s
 => CACHED [build 3/7] COPY go.mod ./                                                                                                                                                                            0.0s
 => CACHED [build 4/7] COPY go.sum ./                                                                                                                                                                            0.0s
 => CACHED [build 5/7] RUN go mod download                                                                                                                                                                       0.0s
 => CACHED [build 6/7] COPY *.go ./                                                                                                                                                                              0.0s
 => CACHED [build 7/7] RUN CGO_ENABLED=0 go build -o /cms-daq-simple-app                                                                                                                                         0.0s
 => CACHED [stage-1 1/2] COPY --from=build /cms-daq-simple-app /cms-daq-simple-app                                                                                                                               0.0s
 => exporting to image                                                                                                                                                                                           0.0s
 => => exporting layers                                                                                                                                                                                          0.0s
 => => writing image sha256:62d498577d24d1e0e248147094e8eb7ea259b1209229e162d06409c71fa1c2b3                                                                                                                     0.0s
 => => naming to docker.io/library/cms-daq-workshop-multi-stage                                                                                                                                                  0.0s
```

After the above command finishes successfully, Docker has successfully built our image and assigned a `cms-daq-workshop-multi-stage` tag to it.

## View Local Images

To list images, run the `docker image ls` command:

```bash
$ docker image ls

REPOSITORY                     TAG       IMAGE ID       CREATED              SIZE
cms-daq-workshop-multi-stage   latest    62d498577d24   About a minute ago   6.37MB
cms-daq-workshop-alpine        latest    80d67b7425d5   35 minutes ago       370MB
cms-daq-workshop-debian        latest    f85afa09b2b4   1 hours ago          865MB
```

As you can see, the final **multi-stage and scratch-based image is significantly smaller** than the Debian-based and the Alpine-based images.

!!! Info
    Note that the final multi-stage image used in this example is about x136 times smaller than the Debian-based image and x60 smaller than the Alpine-based image!

## Run a Container based on the Built Image

After creating the container image, we can instruct Docker to run/start a container, based on the aforementioned image:

```
$ docker run -d --name cms-daq-simple-app-multi-stage \
    -p 8082:8080 cms-daq-workshop-multi-stage

1dc9259eee58273989042a8f8809b55a0ceb1718b90e1c33ee392b2884229875
```

The above command should create a new container named `cms-daq-simple-app-multi-stage` (if it doesn't already exist in your system), based on the `cms-daq-workshop-multi-stage` image on your local machine.

As we're mapping the port `8080` from the container to the port `8082` of our machine, our Golang web application should be reachable from our local machine via:

```
$ curl http://localhost:8082

Hello CMS DAQ Group!
```

Alternatively, [http://localhost:8082/](http://localhost:8082/) should be also accessible from the browser.

## Summary

In this section we went throught:

- multi-stage builds that can be used to reduce the final container image size.
- use a `scratch` base image for our final/production image.

## References

- Docker [multi-stage builds](https://docs.docker.com/build/building/multi-stage/)
- [`scratch` base image](https://hub.docker.com/_/scratch)
