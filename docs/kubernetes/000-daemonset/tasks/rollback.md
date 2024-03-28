# Perform a Rollback on a DaemonSet

This page shows how to perform a rollback on a DaemonSet.

## Prerequisites

The following prerequisites should be met:

- Have the `fluentd-elasticsearch` DaemonSet object applied to your cluster, as described in the initial chapter about DaemonSets.

## Performing a rollback on a DaemonSet

### Step 1: Find the DaemonSet revision you want to roll back to

List all revisions of a DaemonSet:

```bash
kubectl rollout history daemonset <daemonset-name>
```

This returns a list of DaemonSet revisions:

```bash
daemonsets "<daemonset-name>"
REVISION        CHANGE-CAUSE
1               ...
2               ...
...
```

- Change cause is copied from DaemonSet annotation `kubernetes.io/change-cause` to its revisions upon creation.

To see the details of a specific revision:

```bash
kubectl rollout history daemonset <daemonset-name> --revision=1
```

This returns the details of that revision:

```bash
daemonsets "<daemonset-name>" with revision #1
Pod Template:
Labels:       foo=bar
Containers:
app:
 Image:        ...
 Port:         ...
 Environment:  ...
 Mounts:       ...
Volumes:      ...
```

### Step 2: Roll back to a specific revision

```bash
# Specify the revision number you get from Step 1 in --to-revision
kubectl rollout undo daemonset <daemonset-name> --to-revision=<revision>
```

!!! note
    If `--to-revision` flag is not specified, kubectl picks the most recent revision.

### Step 3: Watch the progress of the DaemonSet rollback

`kubectl rollout undo daemonset` tells the server to start rolling back the DaemonSet. The real rollback is done asynchronously inside the cluster control plane.

To watch the progress of the rollback:

```bash
kubectl rollout status ds/<daemonset-name>
```

When the rollback is complete, the output is similar to:

```bash
daemonset "<daemonset-name>" successfully rolled out
```

## Clean up

Delete DaemonSet from a namespace:

```bash
kubectl delete ds fluentd-elasticsearch -n kube-system
```
