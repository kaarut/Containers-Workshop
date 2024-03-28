# Affinity and anti-affinity

`nodeSelector` is the simplest way to constrain Pods to nodes with specific labels. Affinity and anti-affinity expands the types of constraints you can define. Some of the benefits of affinity and anti-affinity include:

- The affinity/anti-affinity language is more expressive. `nodeSelector` only selects nodes with all the specified labels. Affinity/anti-affinity gives you more control over the selection logic.
- You can indicate that a rule is _soft_ or _preferred_, so that the scheduler still schedules the Pod even if it can't find a matching node.
- You can constrain a Pod using labels on other Pods running on the node (or other topological domain), instead of just node labels, which allows you to define rules for which Pods can be co-located on a node.


The affinity feature consists of two types of affinity:

- [**_Node affinity_**](#node-affinity) functions like the `nodeSelector` field but is more expressive and allows you to specify soft rules.
- [**_Inter-pod affinity/anti-affinity_**](#inter-pod-affinity-and-anti-affinity) allows you to constrain Pods against labels on other Pods.

## Node affinity

Node affinity is conceptually similar to `nodeSelector`. There are two types of node affinity:

- `requiredDuringSchedulingIgnoredDuringExecution`: The scheduler **can't schedule the Pod unless the rule is met**. This functions like `nodeSelector`, but with a more expressive syntax.
- `preferredDuringSchedulingIgnoredDuringExecution`: The scheduler **tries to find a node that meets the rule**. If a matching node is not available, the scheduler still schedules the Pod.

!!! note
    In the preceding types, `IgnoredDuringExecution` means that if the node labels change after Kubernetes schedules the Pod, the Pod continues to run.


![Node Affinity Available](../../../img/scheduling/node-affinity-types-available.png)

![Node Affinity Planned](../../../img/scheduling/node-affinity-types-planned.png)

You can specify node affinities using the `.spec.affinity.nodeAffinity` field in your Pod spec:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
  labels:
    app.kubernetes.io/name: nginx
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: topology.kubernetes.io/zone
            operator: In
            values:
            - cern-geneva-a
            - cern-geneva-b
            - cern-geneva-c
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 50
        preference:
          matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
  containers:
  - name: with-node-affinity
    image: nginx:1.23.3
```

In this example, the following rules apply:

- The node **must** have a label with the key `failure-domain.beta.kubernetes.io/zone` and the value of that label must be either `cern-geneva-a`, `cern-geneva-b` or `cern-geneva-c`.
- The node _preferably_ has a label with the key `another-node-label-key` and the value `another-node-label-value`.


You can use the `operator` field to specify a logical operator for Kubernetes to use when interpreting the rules. You can use `In`, `NotIn`, `Exists`, `DoesNotExist`, `Gt` and `Lt`.

`NotIn` and `DoesNotExist` allow you to define node anti-affinity behavior. Alternatively, you can use node taints to repel Pods from specific nodes.

!!! note
    If you specify both `nodeSelector` and `nodeAffinity`, both must be satisfied for the Pod to be scheduled onto a node.

    If you specify multiple terms in `nodeSelectorTerms` associated with `nodeAffinity` types, then the Pod can be scheduled onto a node if one of the specified terms can be satisfied (terms are ORed).

    If you specify multiple expressions in a single `matchExpressions` field associated with a term in `nodeSelectorTerms`, then the Pod can be scheduled onto a node only if all the expressions are satisfied (expressions are ANDed).

### Node affinity weight

You can specify a weight between 1 and 100 for each instance of the `preferredDuringSchedulingIgnoredDuringExecution` affinity type. When the scheduler finds nodes that meet all the other scheduling requirements of the Pod, the scheduler iterates through every preferred rule that the node satisfies and adds the value of the `weight` for that expression to a sum.

The final sum is added to the score of other priority functions for the node. Nodes with the highest total score are prioritized when the scheduler makes a scheduling decision for the Pod.


## Inter-pod affinity and anti-affinity

**Inter-pod affinity and anti-affinity allow you to constrain which nodes your Pods can be scheduled on based on the labels of Pods already running on that node, instead of the node labels**.

Inter-pod affinity and anti-affinity rules take the form "this Pod should (or, in the case of anti-affinity, should not) run in an X if that X is already running one or more Pods that meet rule Y", where X is a _topology domain_ like node, rack, cloud provider zone or region, or similar and Y is the rule Kubernetes tries to satisfy.

You express these rules (Y) as label selectors and the _topology domain_ (X) using a `topologyKey`, which is the key for the node label that the system uses to denote the domain.

!!! note
    Inter-pod affinity and anti-affinity require substantial amount of processing which can slow down scheduling in large clusters significantly. It is not recommended to use them in clusters larger than several hundred nodes.

### Types of inter-pod affinity and anti-affinity

Similar to [node affinity](#node-affinity) are two types of Pod affinity and anti-affinity as follows:

- `requiredDuringSchedulingIgnoredDuringExecution`
- `preferredDuringSchedulingIgnoredDuringExecution`

For example:

- you could use `requiredDuringSchedulingIgnoredDuringExecution` affinity to tell the scheduler to co-locate Pods of two services in the same cloud provider zone because they communicate with each other a lot.
- you could use `preferredDuringSchedulingIgnoredDuringExecution` anti-affinity to spread Pods from a service across multiple cloud provider zones.

### Pod affinity example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-with-pod-affinity
  labels:
    app.kubernetes.io/name: nginx
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: security
            operator: In
            values:
            - S1
        topologyKey: topology.kubernetes.io/zone
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: security
              operator: In
              values:
              - S2
          topologyKey: topology.kubernetes.io/zone
  containers:
  - name: with-pod-affinity
    image: nginx:1.23.3
```

This example defines one Pod affinity rule and one Pod anti-affinity rule. The Pod affinity rule uses the "hard" `requiredDuringSchedulingIgnoredDuringExecution`, while the anti-affinity rule uses the "soft" `preferredDuringSchedulingIgnoredDuringExecution`.

- The affinity rule says that the scheduler can only schedule a Pod onto a node if the node is in the same zone as one or more existing Pods with the label `security=S1`. More precisely, the scheduler must place the Pod on a node that has the `topology.kubernetes.io/zone=V` label, as long as there is at least one node in that zone that currently has one or more Pods with the Pod label `security=S1`.

- The anti-affinity rule says that the scheduler should try to avoid scheduling the Pod onto a node that is in the same zone as one or more Pods with the label `security=S2`. More precisely, the scheduler should try to avoid placing the Pod on a node that has the `topology.kubernetes.io/zone=R` label if there are other nodes in the same zone currently running Pods with the `Security=S2` Pod label.

!!! info
    We'll have a closer look at inter-pod affinity and anti-affinity once we introduce Deployments, where we can have multiple replicas of a Pod.

!!! example
    **Exercise**: Apply the above manifest with the `podAffinity` and `podAntiAffinity` rules. If there is no Pod in your cluster with the label `security=S1`, the new Pod will remain in a `Pending` state. Then, create a new Pod with the `security=S1` label and see if the Pod is being scheduled in any node.
