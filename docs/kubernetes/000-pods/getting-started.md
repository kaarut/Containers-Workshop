# Pods

**Pods are the smallest deployable units of computing that you can create and manage in Kubernetes**. In other words, if you need to run a single container in Kubernetes, then you need to create a Pod for that container.

**A Pod is a group of one or more containers, with shared storage and network resources**, and a specification for how to run the containers.

Containers in a Pod run on a "logical host"; they use the same network namespace (in other words, the same IP address and port space), and the same IPC namespace. They can also use shared volumes. These properties make it possible for these containers to efficiently communicate, ensuring data locality. Also, Pods enable you to manage several tightly coupled application containers as a single unit.

**Containers in a Pod share Linux namespaces (e.g. network and storage)**, but different cgroups.


## "Naked" Pods versus ReplicaSets, Deployments, and Jobs

**Don't use naked Pods** (that is, Pods not bound to a ReplicaSet or Deployment) if you can avoid it. Naked Pods will not be rescheduled in the event of a node failure.

Instead, create them using workload resources such as `Deployment` or `Job`.

A `Deployment`, which both creates a `ReplicaSet` to ensure that the desired number of Pods is always available, and specifies a strategy to replace Pods (such as RollingUpdate), is almost always preferable to creating Pods directly, except for some explicit `restartPolicy: Never` scenarios. A `Job` may also be appropriate.


## Pod Templates

Controllers for workload resources create Pods from a pod template and manage those Pods on your behalf.

PodTemplates are specifications for creating Pods, and are included in workload resources such as `Deployments`, `Jobs`, and `DaemonSets`.
