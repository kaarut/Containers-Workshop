# Dockerfile

Docker can build images automatically by reading the instructions from a `Dockerfile`. A `Dockerfile` is a text document that contains all the commands a user could call on the command line to assemble an image.

A Dockerfile is the building block in the Docker ecosystem. It describes the steps for creating a Docker image.

!!! info
    The (typical) flow of information follows this central model:
    ```
    Dockerfile --> Docker image --> Docker container
    ```

## Format

Here is the format of the `Dockerfile`:

```docker
# Comment
INSTRUCTION arguments
```

**The instruction is not case-sensitive**. However, convention is for them to be UPPERCASE to distinguish them from arguments more easily.

Basically, a Dockerfile is just a normal text file.

**Docker runs instructions in a Dockerfile in order. A Dockerfile must begin with a `FROM` instruction.**

The `FROM` instruction specifies the Parent Image from which you are building.

## Important Docker Instructions

| Instruction | Description | Comment |
:------------:|:-----------:|:-------:|
FROM | Set base image | Must appear as the first instruction; only one entry per build stage |
ENV | Set environment variables for build process and container runtime | — |
ARG | Declare command line parameters for build process | May appear before the FROM instruction |
WORKDIR | Change current directory | — |
USER | Change user and group membership | - |
COPY | Copy files and directories to the image | Creates new layer |
ADD | Copy files and directories to the image | Creates new layer; use is discouraged |
RUN | Execute command in image during build process | Creates new layer |
CMD | Set default arguments for container start | Only one entry per build stage. Creates new layer |
ENTRYPOINT | Set default command for container start | Only one entry per build stage |
EXPOSE | Define port assignments for running container | Ports must be exposed when starting the container |
VOLUME | Include directory in the image as a volume when starting the container in the host system | — |

## Building a Docker Image

A Docker image is created by executing the instructions in a `Dockerfile`. This step is called the build process and is started by executing the `docker build` command.

The _build context_ defines which files and directories the build process has access to.

The instructions in the Dockerfile get access to the files and directories in the build context.

## Resources

- [Docker official docs for `Dockerfile`](https://docs.docker.com/engine/reference/builder/)
