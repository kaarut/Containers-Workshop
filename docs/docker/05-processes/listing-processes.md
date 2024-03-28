# Listing Processes in Running a Container

A container’s main running process is the `ENTRYPOINT` and/or `CMD` at the end of the `Dockerfile`.

It is generally recommended that you separate areas of concern by using one service per container. That service may fork into multiple processes (for example, Nginx starts multiple worker processes).

You can connect multiple containers using user-defined networks and shared volumes.

## Docker Command

The `docker container top` command can be used to display the running processes of a container:

```bash
# Start a container named `test-nginx`
$ docker run -d --name test-nginx nginx:1.23.2-alpine

b1ecdbd23807784169f4788ddf51871ab3438a4fc096e4f84448827fcd068923

# Display running processes of the `test-nginx` container
$ docker container top test-nginx

UID        PID       PPID      C    STIME    TTY    TIME      CMD
root      113986    113967     0    09:44     ?   00:00:00    nginx: master process nginx -g daemon off;
101       114044    113986     0    09:44     ?   00:00:00    nginx: worker process
101       114045    113986     0    09:44     ?   00:00:00    nginx: worker process
```

In the example above, you'll notice that the `test-nginx` container runs the main Nginx master daemon and two worker processes.

## Executing Command inside the Container

The processes of a running container can also be displayed by running the `ps` command against it:

```bash
# Start a container named `test-nginx` (if it's not already running from the previous step)
$ docker run -d --name test-nginx nginx:1.23.2-alpine

# Execute a command to list all processes inside the container
$ docker exec -it test-nginx ps aux

PID   USER     TIME  COMMAND
    1 root      0:00 nginx: master process nginx -g daemon off;
   30 nginx     0:00 nginx: worker process
   31 nginx     0:00 nginx: worker process
   32 root      0:00 ps aux
```

!!!note
    Notice that the PIDs are different between the two approaches:

    - The `docker exec -it test-nginx ps aux` command shows PIDs inside the Docker container.
    - The `docker container top test-nginx` command shows host's system PIDs.

!!!tip
    If [`htop`](https://github.com/htop-dev/htop) is installed in your container, you can view processes in an interactive way, view their CPU and Memory usage.

## References

- [`docker container top` official docs](https://docs.docker.com/engine/reference/commandline/container_top/)
