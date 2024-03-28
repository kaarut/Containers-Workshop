# Assigning Pods to Nodes

You can constrain a Pod so that it is _restricted_ to run on particular node(s), or to _prefer_ to run on particular nodes.

Often, you do not need to set any such constraints; the scheduler will automatically do a reasonable placement (for example, spreading your Pods across nodes so as not place Pods on a node with insufficient free resources). However, there are some circumstances where you may want to control which node the Pod deploys to, for example:

- to ensure that a Pod ends up on a node with an SSD attached to it
- to co-locate Pods from two different services that communicate a lot into the same availability zone
- when there are only two Pods, you'd prefer not to have both of those Pods run on the same node: you would run the risk that a single node failure takes your workload offline.

You can use any of the following methods to choose where Kubernetes schedules specific Pods:

- [`nodeSelector`](./nodeSelector.md) field matching against node labels
- [Affinity and anti-affinity](./affinity-and-anti-affinity.md)
- [`nodeName`](./nodeName.md) field
- [Pod topology spread constraints](./pod-topology-spread-constraints.md)
