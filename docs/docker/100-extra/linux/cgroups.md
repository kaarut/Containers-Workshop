# Cgroups

## Overview

If namespaces are about isolation, control groups (cgroups) are about setting limits.

Containers are isolated from each other but all share a common set of OS resources — things like CPU, RAM, network bandwidth, and disk I/O. Cgroups let us set limits on each of these so a single container cannot consume everything and cause a denial of service (DoS) attack.

## View Containers Resource Usage Statistics

The `docker stats` command is used to display a live stream of container(s) resource usage statistics. Usage:

```bash
docker stats [OPTIONS] [CONTAINER...]
```

The docker stats command returns a live data stream for running containers. To limit data to one or more specific containers, specify a list of container names or ids separated by a space. 

## Container without Resource Limits

1. Create a container without resource limits:

    ```bash
    docker run -d \
        --name my-nginx-container \
        nginx:1.23.2
    ```

1. Display container's resource usage statistics:

    ```bash
    docker stats --no-stream
    ```

## Container with Memory Limit

1. Create a container and set Memory limits with the `--memory` or `-m` flag:

    ```bash
    docker run -d \
        --name my-nginx-container \
        -m 500M \
        nginx:1.23.2
    ```

    Here, we create a container and setting the _maximum_ amount of memory the container can use to be `500M`.

1. Display container's resource usage statistics and verify the upper limit we set in the previous step:

    ```bash
    docker stats --no-stream
    ```

!!! note
    By default, if an out-of-memory (OOM) error occurs, the kernel kills processes in a container.
