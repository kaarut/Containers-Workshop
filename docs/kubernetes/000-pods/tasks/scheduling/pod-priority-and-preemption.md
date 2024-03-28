# Pod Priority and Preemption

Pods can have _priority_. Priority indicates the importance of a Pod relative to other Pods. If a Pod cannot be scheduled, the scheduler tries to preempt (evict) lower priority Pods to make scheduling of the pending Pod possible.

## How to use priority and preemption

To use priority and preemption:

1. Add one or more PriorityClasses.
1. Create Pods with `priorityClassName` set to one of the added PriorityClasses.

!!! note
    The PriorityClass objects are usually provided by the cluster administrators.

!!! note
    Kubernetes already ships with two PriorityClasses: `system-cluster-critical` and `system-node-critical`. These are common classes and are used to ensure that critical components are always scheduled first (e.g. DNS, metrics-server, etc).

## PriorityClass

A PriorityClass is a _non-namespaced object_ that defines a mapping from a priority class name to the integer value of the priority. The value is specified in the required `value` field. **The higher the value, the higher the priority**. The name of a PriorityClass object must be a valid DNS subdomain name, and it cannot be prefixed with `system-`.

## Example PriorityClass

The example below is a PriorityClass object named ``

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "High priority apps"
```

To list the PriorityClasses (remember PriorityClass is a cluster-wide concept and not namespaced):

```bash
$ kubectl get priorityclass

NAME                      VALUE        GLOBAL-DEFAULT   AGE
system-cluster-critical   2000000000   false            90d
system-node-critical      2000001000   false            90d
```

You can get more information about a PriorityClass by either `kubectl describe priorityclass <NAME>` or by `kubectl get priorityclass <NAME> --output yaml` commands.

## Pod priority

After you have one or more PriorityClasses, you can create Pods that specify one of those PriorityClass names in their specifications.

The following manifest is an example of a Pod configuration that uses the PriorityClass created in the preceding example.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-priorityclass-high
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
  priorityClassName: high-priority
```

### Effect of Pod priority on scheduling order

When Pod priority is enabled, the scheduler orders pending Pods by their priority and a pending Pod is placed ahead of other pending Pods with lower priority in the scheduling queue. As a result, the higher priority Pod may be scheduled sooner than Pods with lower priority if its scheduling requirements are met. If such Pod cannot be scheduled, scheduler will continue and tries to schedule other lower priority Pods.

## Preemption

When Pods are created, they go to a queue and wait to be scheduled. The scheduler picks a Pod from the queue and tries to schedule it on a Node. **If no Node is found that satisfies all the specified requirements of the Pod, preemption logic is triggered for the pending Pod**.

Let's call the pending Pod P. Preemption logic tries to find a Node where removal of one or more Pods with lower priority than P would enable P to be scheduled on that Node. If such a Node is found, one or more lower priority Pods get evicted from the Node. After the Pods are gone, P can be scheduled on the Node.

!!! note
    A Node is considered for preemption only when the answer to this question is yes: "If all the Pods with lower priority than the pending Pod are removed from the Node, can the pending Pod be scheduled on the Node?"

    Preemption does not necessarily remove all lower-priority Pods. If the pending Pod can be scheduled by removing fewer than all lower-priority Pods, then only a portion of the lower-priority Pods are removed.

## Limit Priority Class consumption

In a cluster where not all users are trusted, a malicious user could create Pods at the highest possible priorities, causing other Pods to be evicted/not get scheduled.

Cluster administrators are able to restrict usage of certain high priority classes to a limited number of namespaces and not every namespace will be able to consume these priority classes by default (e.g. it may be desired that pods with the `cluster-services` priority, should only be allowed in the `kube-system` namespace).

More details can be found [here](https://kubernetes.io/docs/concepts/policy/resource-quotas/#limit-priority-class-consumption-by-default).
