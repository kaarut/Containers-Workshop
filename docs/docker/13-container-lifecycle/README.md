# Docker Container Lifecycle

## Overview

The complete lifecycle of a Docker container revolves around five phases:

- Create phase
- Running phase
- Paused phase/unpause phase
- Stopped phase
- Killed phase

![Docker Containers Lifecycle](./img/docker-container-lifecycle.png)

## Create Phase

In the create phase, a docker container is created from a docker image.

```bash
docker create --name <NAME> <IMAGE_NAME>
```

## Running Phase

In the running phase, the docker container starts executing the commands mentioned in the image. To run a docker container, use the `docker run` command.

```bash
docker run <CONTAINER_NAME_OR_ID>
```

The `docker run` command creates a container if it is not present. In this case, the creation of the container can be skipped.

## Paused phase

In the paused phase, the current executing command in the docker container is paused. Use the `docker pause` command to pause a running container.

```bash
docker pause container <CONTAINER_NAME_OR_ID>
```

!!!note
    The `docker pause` command pauses all the processes in the container. It sends a `SIGSTOP` signal to pause the processes in the container.


## Unpause phase

In the unpause phase, the paused container resumes executing the commands once it is unpaused.

```bash
docker unpause <CONTAINER_NAME_OR_ID>
```

!!!note
    The `docker unpause` command sends a `SIGCONT` signal to the resume the processes of a paused container.


## Stop phase

In the stop phase, the container’s main process is shutdown gracefully. Docker sends `SIGTERM` for graceful shutdown, and if needed, `SIGKILL`, to kill the container’s main process.

```bash
docker stop <CONTAINER_NAME_OR_ID>
```

## Kill phase

In the kill phase, the container’s main processes are shutdown abruptly. Docker sends a `SIGKILL` signal to kill the container’s main process.

```bash
docker kill <CONTAINER_NAME_OR_ID>
```

## References

- [Docker Internals](http://docker-saigon.github.io/post/Docker-Internals/)
- [What is the Docker container lifecycle?](https://www.educative.io/answers/what-is-the-docker-container-lifecycle)
