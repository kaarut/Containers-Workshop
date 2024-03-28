# Run Command in a Running Container

When a container is up and running, we can run a command against it.


## Docker Exec

The `docker exec` command runs a new command in a running container:

```bash
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

COMMAND will run in the default directory of the container. If the underlying image has a custom directory specified with the `WORKDIR` directive in its Dockerfile, this will be used instead.

COMMAND should be an executable, a chained or a quoted command will not work.

Example: `docker exec -ti my_container "echo a && echo b"` will not work, but `docker exec -ti my_container sh -c "echo a && echo b"` will.

### Examples

1. Create a container:

    ```bash
    docker run --name my-alpine -dit alpine
    ```

    The above command should create a container named `my-alpine`.

1. Execute a command on the `my-alpine` container:

    ```bash
    docker exec -it my-alpine ip a
    ```

    This will display info about all network interfaces in the `my-alpine` container.

1. Next, execute an interactive shell on the container:

    ```bash
    docker exec -it my-alpine sh
    ```

    This will create a new shell session in the container `my-alpine`, where you can interact with shell commands (or install new tools).

### Notes

- By default `docker exec` command runs in the same working directory set when container was created.
- The `docker exec` command will fail on Paused containers.


## Resources

- [`docker exec` official documentation](https://docs.docker.com/engine/reference/commandline/exec/)
