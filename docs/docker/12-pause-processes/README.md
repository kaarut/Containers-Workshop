# Pause and Unpause Processes

## Pause Processes

The `docker pause` command can be used to pause all processes within one or more containers.

The command can be used as follows:

```bash
docker pause CONTAINER [CONTAINER...]
```

The `docker pause` command suspends all processes in the specified containers.

On Linux, this uses the [freezer cgroup](https://www.kernel.org/doc/Documentation/cgroup-v1/freezer-subsystem.txt). Traditionally, when suspending a process the `SIGSTOP` signal is used, which is observable by the process being suspended. With the freezer cgroup the process is unaware, and unable to capture, that it is being suspended, and subsequently resumed.

Let's see this in practice, by starting and then pausing a container:

1. Start a new Docker container named `nginx-to-pause`.

    ```bash
    docker run -d --name nginx-to-pause nginx:1.23.2
    ```

    The above command should create a new Docker container, if the container name doesn't already exist in your system.

1. List the (running) containers:

    ```bash
    $ docker container ls

    CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS     NAMES
    5e669afd726e   nginx:1.23.2   "/docker-entrypoint.…"   2 seconds ago   Up 2 seconds   80/tcp    nginx-to-pause
    ```

    The `nginx-to-pause` container should be in a `Running` state.

1. Pause the running container:

    ```bash
    docker pause nginx-to-pause
    ```

1. List containers:

    ```bash
    $ docker container ls

    CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS                  PORTS     NAMES
    5e669afd726e   nginx:1.23.2   "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes (Paused)   80/tcp    nginx-to-pause
    ```

    The `nginx-to-pause` container should now be reported as Paused.

## Unpause Processes

To unpause all processes, the `docker unpause` command is used.

The `docker unpause` command un-suspends all processes in the specified containers. On Linux, it does this using the freezer cgroup.

For example, to unpause the `nginx-to-pause` container that we paused in the previous paragraph:

```bash
docker unpause nginx-to-pause
```
