# Node Labels

Like many other Kubernetes objects, nodes have [labels](../../../../000-objects/labels-and-selectors.md).

Adding labels to nodes allows you to target Pods for scheduling on specific nodes or groups of nodes. You can use this functionality to ensure that specific Pods only run on nodes with certain isolation, security, or regulatory properties.

## Pre-populated node labels

Kubernetes populates a standard set of labels on all nodes in a cluster. A few well-know labels:

- `kubernetes.io/arch`
- `kubernetes.io/hostname`
- `topology.kubernetes.io/zone`
- `kubernetes.io/os=linux`
- etc.

Kubernetes reserves all labels and annotations in the `kubernetes.io` and `k8s.io` namespaces.

A more detailed list of well-known labels, annotations and taints cane be found [here](https://kubernetes.io/docs/reference/labels-annotations-taints/).

### Manually attaching node labels

You can also attach labels _manually_:

1. List the nodes in your cluster, along with their labels:

    ```bash
    kubectl get nodes --show-labels
    ```

    The output is similar to this:

    ```bash
    NAME      STATUS    ROLES    AGE     VERSION        LABELS
    worker0   Ready     <none>   1d      v1.13.0        ...,kubernetes.io/hostname=worker0
    worker1   Ready     <none>   1d      v1.13.0        ...,kubernetes.io/hostname=worker1
    worker2   Ready     <none>   1d      v1.13.0        ...,kubernetes.io/hostname=worker2
    ```

1. Choose one of your nodes, and add a label to it:

    ```bash
    kubectl label nodes <your-node-name> disktype=ssd
    ```

    where `<your-node-name>` is the name of your chosen node.

1. Verify that your chosen node has a disktype=ssd label:

    ```bash
    kubectl get nodes --show-labels
    ```

    The output is similar to this:

    ```bash
    NAME      STATUS    ROLES    AGE     VERSION        LABELS
    worker0   Ready     <none>   1d      v1.13.0        ...,disktype=ssd,kubernetes.io/hostname=worker0
    worker1   Ready     <none>   1d      v1.13.0        ...,kubernetes.io/hostname=worker1
    worker2   Ready     <none>   1d      v1.13.0        ...,kubernetes.io/hostname=worker2
    ```

    In the preceding output, you can see that the `worker0` node has a `disktype=ssd label`.

!!! info
    The node labels are usually handled by cluster administrators.
