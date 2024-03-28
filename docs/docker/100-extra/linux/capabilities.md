# Linux Capabilities

## Overview

It’s a bad idea to run containers as `root` — `root` is all-powerful and therefore very dangerous. But, it can be challenging running containers as unprivileged non-root users. For example, on most Linux systems, non-root users tend to be so powerless they’re practically useless.

_Capabilities_ is a technology that lets us pick and choose which root powers a container needs in order to run.

Under the hood, the Linux root user is a combination of a long list of capabilities. Some of these capabilities include:

- `CAP_CHOWN`: lets you change file ownership
- `NET_RAW`: lets you use RAW and PACKET sockets
- `CAP_NET_BIND_SERVICE`: lets you bind a socket to low numbered network ports
- `CAP_SETUID`: lets you elevate the privilege level of a process
- `CAP_SYS_BOOT`: lets you reboot the system.

## Example

Let's illustrate an example of dropping specific Linux capabilities.

In this example we're going to observe how the `ping` command behaves inside a container with and without the `NET_RAW` capability.

For each scenario we are going to:
- create an Ubuntu-based container
- install the required packages for this exercise
- ping the local host

### Container with All Capabilities

1. Create an Ubuntu-based container and get an interactive `bash` shell:

    ```bash
    docker run --rm -it ubuntu:22.10 bash
    ```

1. Install packages **inside the container**:

    ```bash
    apt-get update -y && \
    apt-get install -y iputils-ping
    ```

1. Ping the local host:

    ```bash
    $ ping 127.0.0.1 -c 2

    PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
    64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.021 ms
    64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.028 ms

    --- 127.0.0.1 ping statistics ---
    2 packets transmitted, 2 received, 0% packet loss, time 1014ms
    rtt min/avg/max/mdev = 0.021/0.024/0.028/0.003 ms
    ```

    As you can see, we are able to use `ping` and reach the local host.

1. Exit the interactive `bash` shell

    ```bash
    root@fbfdbce28eb0:/  exit
    ```

The container should be automatically removed, as we used the `--rm` flag when creating it.

### Container with Dropped Capabilities

Let's now create a new container but this time with the `NET_RAW` capability dropped and follow the same steps as above.

1. Create an Ubuntu-based container and get an interactive `bash` shell:

    ```bash
    docker run --rm -it --cap-drop=NET_RAW ubuntu:22.10 bash
    ```

1. Install packages **inside the container**:

    ```bash
    apt-get update -y && \
    apt-get install -y iputils-ping
    ```

1. Ping the local host:

    ```bash
    $ ping 127.0.0.1 -c 2

    bash: /usr/bin/ping: Operation not permitted
    ```

    As you can see, we aren't able to use `ping` command this time.

1. Exit the interactive `bash` shell

    ```bash
    root@fbfdbce28eb0:/  exit
    ```

The container should be automatically removed, as we used the `--rm` flag when creating it.


!!! note
    Ubuntu 22.10 has two implementations of `ping`:

    - The default one, from `iputils-ping`, which _might_ require the `CAP_NET_RAW` capability to be executed.
    - The one from `inetutils-ping` which doesn't require the `CAP_NET_RAW` capability.

    More info [here](https://manpages.debian.org/stable/iputils-ping/ping.8.en.html#SECURITY) and [here](https://manpages.debian.org/bullseye/inetutils-ping/ping.1.en.html).

## Home Practise

Create an Ubuntu-based container that drops the `CHOWN` capability. Once created, try to use `chown` to change ownership for a file.

<details>
    <summary>Hint</summary>

Some commands that might come handy (inside the container):

```
$ groupadd -r postgres && useradd --no-log-init -r -g postgres postgres

$ cat /etc/passwd | grep -i postgres

$ ls -al

$ chown postgres .dockerenv
```

</details>

## References

- [Docker Deep Dive (book)](https://learning.oreilly.com/library/view/docker-deep-dive/9781800565135/)
