# Docker Networking Commands

In this section we are going through the main Docker networking commands.

## List Docker Networks

To list all available Docker networks:

```bash
$ docker network ls

NETWORK ID     NAME      DRIVER    SCOPE
bdbbde940ad2   bridge    bridge    local
d554f0d8acb1   host      host      local
8d567bad69e5   none      null      local
```

All Docker installations should be shipped with the default `bridge` network.

If you don't specify any network when creating a new container with `docker run`, the default `bridge` network will be used.

## Inspect Network

To display detailed information on one or more networks:

```bash
$ docker network inspect bridge

[
    {
        "Name": "bridge",
        "Id": "bdbbde940ad2eb64209bfffdc876489d0f19b63a7925614fc469ff5ac54ab4fa",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Containers": {
            "d2d6dfe5398805f62b978b61fe4738babc5f6e9ea35b147b52fdb2d247b6b780": {
                "Name": "my-nginx-container",
                "EndpointID": "4d512c3290727893ac95904f75c1b7f8b233d24e7d4ac6057cb283a376a0e5c3",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        }
    }
]
```

## Get the Networks that a Container is connected to

To get the Docker networks that a container is connected to (make sure to replace `CONTAINER` with either the container name or the container ID):

```bash
docker container inspect \
    --format='{{range $netName, $value := .NetworkSettings.Networks}}{{println $netName}}{{end}}' \
    CONTAINER
```

## Create User-defined bridge Networks

1. To create the `my-bridge-network` network based on on the bridge driver:

    ```bash
    docker network create --driver bridge my-bridge-network
    ```

1. Create two containers - one running Nginx and one that will perform a simple HTTP GET request.

    Let's create the Nginx container first:

    ```bash
    $ docker run -d \
        --network my-bridge-network \
        --name nginx-in-user-defined-network \
        nginx:1.23.2
    ```

    And then create the second container that will run in the same Docker network as the original container:

    ```bash
    docker run -it \
        --network my-bridge-network \
        curlimages/curl \
        http://nginx-in-user-defined-network
    ```

    The last command should return the default Nginx webpage.

## Connect Running Container to a Network

Sometimes it might be useful to connect a running container to a Docker network (for example if we forgot to specify the network when creating the container).

1. Create a container named `my-alpine` in the (default) `bridge` network:
    
    ```bash
    docker run -dit \
        --name my-alpine \
        alpine
    ```

1. To connect the `my-alpine` container to the `my-bridge-network` network, run the following command from shell in your host machine:

    ```bash
    docker network connect my-bridge-network my-alpine
    ```

    !!!note
        We're assuming that the Docker network named `my-bridge-network` has been created in a previous section.

## Find all Containers connected to a Network

To get all containers that are connected to a Docker network (make sure you replace `NETWORK` with network name):

```bash
docker network inspect \
    --format='{{range .Containers}}{{println .Name}}{{end}}' \
    NETWORK
```

## Remove a Docker Network

To remove one or more networks, the `docker network rm` command is used:

```
docker network rm NETWORK [NETWORK...]
```

!!!info
    When removing a Docker network that has containers connected to it, an error message like the following should appear:
    ```
    Error response from daemon: error while removing network:
    network my-bridge-network id 1df2af7887052c9a9a has active endpoints
    ```

    Therefore, to remove a network, you must first disconnect any containers connected to it.
