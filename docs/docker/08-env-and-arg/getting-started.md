# Environment variables and Build Arguments

As we've already seen, the `Dockerfile` is a script containing instructions on how to build a Docker image. Conversely, a Docker container is a runnable instance of an image. Depending on our needs, we may need to have build-time or run-time environment variables.

When using Docker, we distinguish between two different types of variables - ARG and ENV.

## ARG and ENV Availability

**`ARG` are also known as [build-time variables](https://docs.docker.com/engine/reference/builder/#arg)**. They are only available from the moment they are "announced" in the Dockerfile with an `ARG` instruction up to the moment when the image is built. Running containers can’t access values of ARG variables.

**We can access `ENV` values during the build, as well as once the container runs**. However, unlike `ARG`, they are also accessible by containers started from the final image**. **ENV values can be overridden when starting a container.

Here is a simplified overview of ARG and ENV availabilities around the process around building a Docker image from a Dockerfile, and running a container. They overlap, but ARG is not usable from inside the containers.

![ARG and ENV Availability](./docker-env-and-build-args-overview.png)
