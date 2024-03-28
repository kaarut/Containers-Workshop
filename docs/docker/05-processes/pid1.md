# PID 1

## Overview

Running Normally, when you launch a Docker container, the process you're executing becomes PID 1, giving it the quirks and responsibilities that come with being the init system for the container.
Docker containers spawns processes with the PID of 1. If you run your container process wrapped in a shell script, this shell script will be PID 1 and will not pass along any signals to your child process. This means that `SIGTERM`, the signal used for graceful shutdown, will be ignored by your process.

## Details

Normally, when you launch a Docker container, the process you're executing becomes PID 1, giving it the quirks and responsibilities that come with being the init system for the container.

There are two common issues this presents:

1. In most cases, signals won't be handled properly.

    The Linux kernel applies special signal handling to processes which run as PID 1.

    When processes are sent a signal on a normal Linux system, the kernel will first check for any custom handlers the process has registered for that signal, and otherwise fall back to default behavior (for example, killing the process on `SIGTERM`).

    However, if the process receiving the signal is PID 1, it gets special treatment by the kernel; if it hasn't registered a handler for the signal, the kernel won't fall back to default behavior, and nothing happens. In other words, if your process doesn't explicitly handle these signals, sending it `SIGTERM` will have no effect at all.

    A common example is CI jobs that do `docker run my-container script`: sending `SIGTERM` to the `docker run` process will typically kill the `docker run` command, but leave the container running in the background.

1. Orphaned zombie processes aren't properly reaped.

    A process becomes a zombie when it exits, and remains a zombie until its parent calls some variation of the `wait()` system call on it. It remains in the process table as a "defunct" process. Typically, a parent process will call `wait()` immediately and avoid long-living zombies.

    If a parent exits before its child, the child is "orphaned", and is re-parented under PID 1. The init system is thus responsible for `wait()`-ing on orphaned zombie processes.

    Of course, most processes won't `wait()` on random processes that happen to become attached to them, so containers often end with dozens of zombies rooted at PID 1.

## Send a Custom Signal to a Docker Container

You can use the `docker kill` command to kill one or more running containers. The full command is:

```bash
docker kill [OPTIONS] CONTAINER [CONTAINER...]
```

For example, to send a `SIGKILL` signal to a running container:

```bash
docker kill --signal=SIGKILL  MY_CONTAINER
```

## Extras

There are a few minimal init system for Linux containers that run as PID 1 inside containers, spawn your commands as child processes, handle and forward signals as they are received. Some popular tools include:

- [dumb-init](https://github.com/Yelp/dumb-init)
- [tini](https://github.com/krallin/tini)

## References

- [PID 1 Signal Handling in Docker](https://petermalmgren.com/signal-handling-docker/)
- [`dumb-init` docs](https://github.com/Yelp/dumb-init)
