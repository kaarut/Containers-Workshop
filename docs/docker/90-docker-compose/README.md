# Docker Compose

The Compose file is a YAML file defining services, networks, and volumes for a Docker application.

## Overview

In short, Docker Compose works by applying many rules declared within a single `docker-compose.yaml` configuration file.

These YAML rules, both human-readable and machine-optimized, provide an effective way to snapshot the whole project in a few lines.

Almost every rule replaces a specific Docker command, so that in the end, we just need to run:

```bash
docker compose up
```

## Services

Computing components of an application are defined as Services. A Service is an abstract concept implemented on platforms by running the same container image (and configuration) one or more times.

### Pulling an image

Sometimes, the image we need for our service has already been published to a remote registry.

If that's the case, then we refer to it with the image attribute by specifying the image name and tag:

```yaml
services:
  my-service:
    image: ubuntu:latest
    ...
```

### Building an Image

Alternatively, we might need to build an image from the source code by reading its Dockerfile.

This time, we'll use the build keyword, passing the path to the Dockerfile as the value:

```yaml
services:
  my-custom-app:
    build: /path/to/dockerfile/
    ...
```

### Declaring the Dependencies

Often, we need to create a dependency chain between our services so that some services get loaded before (and unloaded after) other ones. We can achieve this result through the `depends_on` keyword:

```yaml
services:
  kafka:
    image: bitnami/kafka:latest
    depends_on:
      - zookeeper
    ...
  zookeeper:
    image: bitnami/zookeeper:latest
    ...
```

### Environment Variables

#### Set environment variables in container

You can set environment variables in a service’s containers with the `environment` key, just like with `docker run -e VARIABLE=VALUE ...`:

```yaml
web:
  environment:
    - DEBUG=1
```

#### The "env_file" configuration option

You can pass multiple environment variables from an external file through to a service’s containers with the `env_file` option, just like with `docker run --env-file=FILE ...`:

```yaml
web:
  env_file:
    - web-variables.env
```

#### Set environment variables with ‘docker compose run’

Similar to `docker run -e`, you can set environment variables on a one-off container with `docker compose run -e`:

```bash
docker compose run -e DEBUG=1 web python console.py
```

## Networks

Services communicate with each other through Networks. In this specification, a Network is a platform capability abstraction to establish an IP route between containers within services connected together. Low-level, platform-specific networking options are grouped into the Network definition and MAY be partially implemented on some platforms.

To define networks in the compose file:

```yaml
networks:
  # The presence of these objects is sufficient to define them
  front-tier: {}
  back-tier: {}
```

To add services in a network:

```yaml
services:
  frontend:
    image: awesome/webapp
    ports:
      - "443:8043"
    networks:
      - front-tier
      - back-tier

  backend:
    image: awesome/database
    networks:
      - back-tier
```

## Volumes

Services store and share persistent data into Volumes. The specification describes such a persistent data as a high-level filesystem mount with global options. Actual platform-specific implementation details are grouped into the Volumes definition and MAY be partially implemented on some platforms.

## Example

A complete example of a `docker-compose.yaml` file looks like the following:

```yaml
networks:
  my-network-name:
    driver: bridge

services:
  zookeeper:
    image: bitnami/zookeeper:latest
    networks:
      - my-network-name
    environment:
      - ALLOW_ANONYMOUS_LOGIN=yes
    deploy:
      restart_policy:
        condition: on-failure
      resources:
        limits:
          cpus: '0.50'
          memory: 500M

  kafka:
    image: bitnami/kafka:latest
    networks:
      - my-network-name
    environment:
      - KAFKA_BROKER_ID=1
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://127.0.0.1:9092
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
      - ALLOW_PLAINTEXT_LISTENER=yes
    depends_on:
      - zookeeper
```

## Lifecycle and Commands

### Startup

We've seen that we can create and start the containers, the networks, and the volumes defined in the configuration with:

```bash
docker compose up
```

After the first time, however, we can simply start the services:

```bash
docker compose start
```

If our file has a different name than the default one (`docker-compose.yaml` or `docker-compose.yml`), we can use the `-f` flags to specify an alternate file name:

```bash
docker compose -f custom-compose-file.yml start
```

Compose can also run in the background as a daemon when launched with the `-d` option:

```bash
docker compose up -d
```

### Shutdown

To safely stop the active services, we can use the `stop` subcommand, which will preserve containers, volumes, and networks, along with every modification made to them:

```bash
docker compose stop
```

To reset the status of our project, we can simply run down, which will destroy everything with the exception of external volumes:

```bash
docker compose down
```
