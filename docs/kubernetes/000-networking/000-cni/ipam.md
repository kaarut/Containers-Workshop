
# IPAM

Pods are best thought of as ephemeral. Specifically, they are prone to being restarted or rescheduled based on the needs of the cluster or system failure. This requires IP allocation to execute quickly and the management of the cluster’s IP pool to be efficient. This management is often referred to as IP Address Management (IPAM) and is not unique to Kubernetes.

**IPAM is implemented based on your choice of CNI plug-in**. There are a few commonalities in these plug-ins that pertain to Pod IPAM.

In the case of boostrapping the cluster with `kubeadm`, a flag regarding the  Pod network’s [Classless Inter-Domain Routing (CIDR)](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) can be passed (e.g. `--pod-network-cidr` flag).

Whether these values are used in IPAM is up to the CNI plug-in. For example, Calico detects and respects this setting, while Cilium offers an option to either manage IP pools independent of Kubernetes (default) or respect these allocations. In most CNI implementations, it is important that **your CIDR choice does not overlap with the cluster’s host/node network**.

!!! note
    How large you should set your cluster’s Pod CIDR is often a product of your networking model. In most deployments, a Pod network is entirely internal to the cluster. As such, the Pod CIDR can be very large to accommodate for future scale. When the Pod CIDR is routable to the larger network, thus consuming address space, you may have to do more careful consideration. Multiplying the number of Pods per node by your eventual node count can give you a rough estimate. The number of Pods per node is configurable on the kubelet, but by default is 110.
