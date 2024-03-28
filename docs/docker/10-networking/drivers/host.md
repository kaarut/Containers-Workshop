# Host Network

If you run containers on a bridge network, they are isolated from other bridge networks and the host.

When you run a container on the host network, it’s able to see everything going on on the host’s network. Essentially, you are skipping network isolation for that container.

This also means, that publishing ports when running on the host network is pointless. The services your container launches bind to the ports on the host interface already.
