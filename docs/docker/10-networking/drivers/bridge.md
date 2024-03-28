# Bridge Networks

In terms of networking, a bridge network is a Link Layer device which forwards traffic between network segments. A bridge can be a hardware device or a software device running within a host machine’s kernel.

In terms of Docker, a bridge network uses a software bridge which allows containers connected to the same bridge network to communicate, while providing isolation from containers which are not connected to that bridge network. The Docker bridge driver automatically installs rules in the host machine so that containers on different bridge networks cannot communicate directly with each other.

**Bridge networks apply to containers running on the same Docker daemon host**. For communication among containers running on different Docker daemon hosts, you can either manage routing at the OS level, or you can use an [overlay network](https://docs.docker.com/network/overlay/).

## Default Docker bridge Network

When you start Docker, a default bridge network (also called `bridge`) is created automatically, and newly-started containers connect to it unless otherwise specified. You can also create user-defined custom bridge networks. **User-defined bridge networks are superior to the default `bridge` network**.

## Differences between user-defined bridges and the default bridge

Some of the _main_ differences between user-defined bridges and the default `bridge` network:

- **User-defined bridges provide automatic DNS resolution between containers**.

    Containers on the default bridge network can only access each other by IP addresses.

    On a user-defined bridge network, containers can resolve each other by name or alias.

- **User-defined bridges provide better isolation**.

    All containers without a `--network` specified, are attached to the default `bridge` network. This can be a risk, as unrelated stacks/services/containers are then able to communicate.

For a more detailed list of differences view the [corresponding section on the official Docker docs](https://docs.docker.com/network/bridge/#differences-between-user-defined-bridges-and-the-default-bridge).

## Docker Compose and Bridge Networks

By default, a new bridge network is created for each docker-compose stack. All containers running in the stack are attached to the same bridge network, and they can reach each other.
