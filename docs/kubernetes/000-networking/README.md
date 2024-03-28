# Networking

There are 4 distinct networking problems to address:

- **Local** communications between containers in the same Pod – solved by the local loopback interface.
- **Pod-to-Pod** East-West communication – solved by a CNI plugin.
- Multi-pod **service** abstraction – a way to group similar Pods and load-balance traffic to them.
- **Ingress** & Egress communication – getting the traffic in and out of the Kubernetes cluster.


In addition to the above, there are a number of auxiliary problems that are covered in their separate chapters:

- **Network Policies** – a way to filter traffic going to and from Pods.
- **DNS** – the foundation of cluster service discovery.
- **IPv6**.

## The Kubernetes network model - High level overview

- **Every Pod in a cluster gets its own unique cluster-wide IP address**. This means you do not need to explicitly create links between Pods and you almost never need to deal with mapping container ports to host ports.

- Kubernetes imposes the following fundamental requirements on any networking implementation (barring any intentional network segmentation policies):
    - pods can communicate with all other pods on any other node without NAT.
    - agents on a node (e.g. system daemons, kubelet) can communicate with all pods on that node.

- **Kubernetes IP addresses exist at the Pod scope** - containers within a Pod share their network namespaces - including their IP address and MAC address.
    - This means that containers within a Pod can all reach each other's ports on localhost.
    - This also means that containers within a Pod must coordinate port usage, but this is no different from processes in a VM.


## How to implement the Kubernetes network model

The network model is implemented by the container runtime on each node. The most common container runtimes use **Container Network Interface** (**CNI**) plugins to manage their network and security capabilities.
