# Docker Network Drivers
## Network Drivers

Docker’s networking subsystem is pluggable, using drivers. Several drivers exist by default, and provide core networking functionality. Some of the most important ones are:

- `bridge` (default): Bridge networks are usually used when your applications run in standalone containers that need to communicate.

- `host`: For standalone containers, remove network isolation between the container and the Docker host, and use the host’s networking directly.

- `overlay`: Overlay networks connect multiple Docker daemons together and enable swarm services to communicate with each other.

Some other popular default network drivers include: `ipvlan`, `macvlan`, `none`. You can also install and use third-party network plugins with Docker.

## Iptables

!!! info
    This section does not go into OS-specific details about how Docker networks work, so you will not find information about how Docker manipulates `iptables` rules on Linux. For more details on Docker and `iptables` see [here](https://docs.docker.com/network/iptables/).

In short, Docker manipulates `iptables` rules on Linux to provide network isolation.

## Network Drivers Summary

- **User-defined bridge networks** are best when you need multiple containers to communicate on the same Docker host.

- **Host networks** are best when the network stack should not be isolated from the Docker host, but you want other aspects of the container to be isolated.

- **Overlay networks** are best when you need containers running on different Docker hosts to communicate, or when multiple applications work together using swarm services.

- **Macvlan networks** are best when you are migrating from a VM setup or need your containers to look like physical hosts on your network, each with a unique MAC address.

- **Third-party network plugins** allow you to integrate Docker with specialized network stacks.
