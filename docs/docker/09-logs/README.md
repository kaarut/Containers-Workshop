# View Logs for a Container

## Overview 

The `docker logs` command shows information logged by a running container. The information that is logged and the format of the log depends almost entirely on the container’s endpoint command.

By default, `docker logs` shows the command’s output just as it would appear if you ran the command interactively in a terminal. UNIX and Linux commands typically open three I/O streams when they run, called `STDIN`, `STDOUT`, and `STDERR`. `STDIN` is the command’s input stream, which may include input from the keyboard or input from another command. `STDOUT` is usually a command’s normal output, and `STDERR` is typically used to output error messages. By default, `docker logs` shows the command’s `STDOUT` and `STDERR`.

## Example

1. Create an Nginx container:

    ```
    $ docker run -d \
        -p 8080:80 \
        --name my-nginx-container \
        nginx:1.23.2

    d2d6dfe5398805f62b978b61fe4738babc5f6e9ea35b147b52fdb2d247b6b78
    ```

1. View the logs for the created container:

    ```bash
    $ docker logs my-nginx-container

    /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
    /docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
    /docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
    10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
    10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
    /docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
    /docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
    /docker-entrypoint.sh: Configuration complete; ready for start up
    2022/11/21 11:51:46 [notice] 1#1: using the "epoll" event method
    2022/11/21 11:51:46 [notice] 1#1: nginx/1.23.2
    2022/11/21 11:51:46 [notice] 1#1: built by gcc 10.2.1 20210110 (Debian 10.2.1-6)
    2022/11/21 11:51:46 [notice] 1#1: OS: Linux 5.10.124-linuxkit
    2022/11/21 11:51:46 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
    2022/11/21 11:51:46 [notice] 1#1: start worker processes
    2022/11/21 11:51:46 [notice] 1#1: start worker process 28
    2022/11/21 11:51:46 [notice] 1#1: start worker process 29
    2022/11/21 11:51:46 [notice] 1#1: start worker process 30
    2022/11/21 11:51:46 [notice] 1#1: start worker process 31
    ```

    !!! tip
        Use the `--follow` or `-f` flag to follow the log output.

## Find Log Path

Docker by default store logs to one log file. To check log file path run command:

```bash
$ docker inspect --format='{{.LogPath}}' my-nginx-container

/var/lib/docker/containers/d2d6dfe5398805f62b978b61fe4738babc5f6e9ea35b147b52fdb2d247b6b78/d2d6dfe5398805f62b978b61fe4738babc5f6e9ea35b147b52fdb2d247b6b78-json.log
```

To see live logs you can run below command:

```bash
tail -f `docker inspect --format='{{.LogPath}}' my-nginx-container`
```
