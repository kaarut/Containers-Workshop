# Listing Docker Containers

## Overview

Docker provides various options to list and filter containers in different states or even options to customize the list output.

## Listing Containers

**In order to list the Docker containers, we can use the `docker ps` or `docker container ls` command**. This command provides a variety of ways to list and filter all containers on a particular Docker engine.

### Aliases

**As of [Docker 1.13](https://www.docker.com/blog/whats-new-in-docker-1-13/), the Docker team regrouped every command to sit under the logical object it’s interacting with**. For instance, in order to list Docker containers, in addition to `docker ps`, we can use the `docker container list` or even `docker container ls` command.

All of these three aliases are supporting the same group of options. However, it's a good idea to adopt the new syntax.

### Running Containers

The `docker container ls` command with no options will _only_ list the _running_ containers:

```bash
$ docker container ls -a

CONTAINER ID   IMAGE           COMMAND                  CREATED          STATUS            PORTS          NAMES
6dd455738e04   nginx:1.23.2    "/docker-entrypoint.…"   24 seconds ago   Up 23 seconds     80/tcp    romantic_feistel
6ff707de3d08   nginx:1.23.2    "/docker-entrypoint.…"   25 seconds ago   Up 25 seconds     80/tcp    determined_feynman
d33e4c445d54   nginx:1.23.2    "/docker-entrypoint.…"   28 seconds ago   Up 28 seconds     80/tcp    interesting_jang
```

We have three running Nginx containers so far.

By default, the output shows several details about each running container:

- `CONTAINER ID` is the container unique identifier. This identifier is the truncated version of a long SHA-256 hash.
- `IMAGE` is the container image name and its tag separated by a colon, such as `nginx:1.23.2`.
- `COMMAND` is the command responsible for running the container.
- `CREATED` shows when the container was created.
- `STATUS` shows the container status. As mentioned above, all these containers are running.
- `PORTS` shows the port mappings between the host machine and inside the container.
- `NAMES` represents the human-readable name of the Docker container, such as `determined_feynman`.

### All Containers

By default, the `docker container ls` command only shows the running containers.

However, if we pass the `-a` option (or `--all`), it'll list all (stopped and running) containers.

### Latest Containers

To see the last _n_ Docker containers (both running and stopped), we can use the `-n <number>` or `–last <number>` option:

```bash
$ docker container ls -n 2

CONTAINER ID   IMAGE                  COMMAND                  CREATED       STATUS       PORTS                  NAMES
a62c3b3a457a   nginx:1.23.2           "/docker-entrypoint.…"   3 hours ago   Up 3 hours   0.0.0.0:8080->80/tcp   my-nginx-container
aef7723ff680   kindest/node:v1.25.2   "/usr/local/bin/entr…"   4 weeks ago   Up 2 days                           kind-worker
```

It's also possible to see the latest container via the `-l` or the `--latest` options:

```bash
$ docker container ls -l

CONTAINER ID   IMAGE          COMMAND                  CREATED       STATUS       PORTS                  NAMES
a62c3b3a457a   nginx:1.23.2   "/docker-entrypoint.…"   3 hours ago   Up 3 hours   0.0.0.0:8080->80/tcp   my-nginx-container
```

### Container Size

We can see the size of a container and its image on disk via the `-s` or `--size` options:

```bash
$ docker container ls --latest -s

CONTAINER ID   IMAGE          COMMAND                  CREATED       STATUS       PORTS                  NAMES                SIZE
a62c3b3a457a   nginx:1.23.2   "/docker-entrypoint.…"   3 hours ago   Up 3 hours   0.0.0.0:8080->80/tcp   my-nginx-container   1.09kB (virtual 135MB)
```

### Customized Output

If we're not happy with the default output format, we can customize the output using the [Go templates](https://golang.org/pkg/text/template/). This can be achieved by passing desired format to the `--format` option:

```bash
$ docker container ls --format "{{.ID}} -> Image: {{.Image}}, named {{.Names}}, ({{.Status}})"

a62c3b3a457a -> Image: nginx:1.23.2, named my-nginx-container, (Up 3 hours)
aef7723ff680 -> Image: kindest/node:v1.25.2, named kind-worker, (Up 2 days)
d3033335d80c -> Image: kindest/node:v1.25.2, named kind-control-plane, (Up 2 days)
d2766818e924 -> Image: kindest/node:v1.25.2, named kind-worker3, (Up 2 days)
5a2b111e28e7 -> Image: kindest/node:v1.25.2, named kind-worker2, (Up 2 days)
```

### Filtering

In order to filter the containers, we can use the `-f` or `--filter` option:

```bash
$ docker container ls --filter "name=my-nginx-container"

CONTAINER ID   IMAGE          COMMAND                  CREATED       STATUS       PORTS                  NAMES
a62c3b3a457a   nginx:1.23.2   "/docker-entrypoint.…"   3 hours ago   Up 3 hours   0.0.0.0:8080->80/tcp   my-nginx-container
```

More details about filtering on the [official Docker documentation](https://docs.docker.com/engine/reference/commandline/ps/#filtering
).
