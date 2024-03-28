# Network Policy

## Problem Outline

**Kubernetes’ default behavior is to allow traffic between any two pods in the cluster network**. This behavior is a deliberate design choice for ease of adoption and flexibility of configuration, but it might be undesirable in practice.

Since all pods can communicate with all other pods, it is recommended that application owners use `NetworkPolicy` objects along with other application-layer security measures, such as authentication tokens or mutual Transport Layer Security (mTLS), for any network communication.

## NetworkPolicy

If you want to **control traffic flow at the IP address or port level (OSI layer 3 or 4)**, then you might consider using Kubernetes **NetworkPolicies** for particular applications in your cluster.

**They are used to control the traffic in(ingress) and out(egress) of pods**.

The entities that a Pod can communicate with are identified through a combination of the following 3 identifiers:

1. Other pods that are allowed (exception: a pod cannot block access to itself)
1. Namespaces that are allowed
1. IP blocks (exception: traffic to and from the node where a Pod is running is always allowed, regardless of the IP address of the Pod or the node)


## Prerequisites

**Network policies are implemented by the network plugin**. To use network policies, you must be using a networking solution which supports NetworkPolicy.

## Practice

[ahmetb/kubernetes-network-policy-recipes, Github](https://github.com/ahmetb/kubernetes-network-policy-recipes)
