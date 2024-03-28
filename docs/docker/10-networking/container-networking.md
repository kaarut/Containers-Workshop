# Container Networking

## Published Ports

By default, when you create or run a container using `docker create` or `docker run`, **it does not publish any of its ports to the outside world**. To make a port available to services outside of Docker, or to Docker containers which are not connected to the container’s network, use the `--publish` or `-p` flag. **This creates a firewall rule which maps a container port to a port on the Docker host to the outside world**. Here are some examples:

| Flag value | Description |
|:----------:|:-----------:|
| `-p 8080:80` | Map TCP port `80` in the container to port `8080` on the Docker host. |
| `-p 192.168.1.100:8080:80` | Map TCP port `80` in the container to port `8080` on the Docker host for connections to host IP `192.168.1.100`. |
| `-p 8080:80/udp` | Map UDP port `80` in the container to port `8080` on the Docker host. |
| `-p 8080:80/tcp -p 8080:80/udp` | Map TCP port `80` in the container to TCP port `8080` on the Docker host, and map UDP port `80` in the container to UDP port `8080` on the Docker host. |

To display all NAT iptables rules in your Linux host:

```bash
iptables -t nat -L -n -v
```

If you publish ports with the `-p` flag, you should be able to spot the iptables rules created by Docker.

## IP Address and Hostname

By default, the container is assigned an IP address for every Docker network it connects to. The IP address is assigned from the pool assigned to the network, so the Docker daemon effectively acts as a DHCP server for each container. Each network also has a default subnet mask and gateway.

When the container starts, it can only be connected to a single network, using `--network`.

In the same way, a container’s hostname defaults to be the container’s ID in Docker. You can override the hostname using `--hostname`.

## DNS Services

By default, a container inherits the DNS settings of the host, as defined in the `/etc/resolv.conf` configuration file. Containers that use the default `bridge` network get a copy of this file, whereas containers that use a custom network use Docker’s embedded DNS server, which forwards external DNS lookups to the DNS servers configured on the host.
