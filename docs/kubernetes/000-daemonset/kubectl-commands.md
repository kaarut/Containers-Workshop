# Kubectl Commands

The following sections provide some usuful `kubectl` commands when working with Daemonset resources:

| Command | Description |
|:-------:|:-----------:|
| `kubectl get daemonset` | List one or more daemonsets |
| `kubectl edit daemonset <daemonset_name>` | Edit and update the definition of one or more daemonset |
| `kubectl describe ds <daemonset_name> -n <namespace_name>` | Display the detailed state of daemonsets within a namespace |
| `kubectl rollout history daemonset <daemonset-name>` | List all revisions of a DaemonSet |
| `kubectl rollout history daemonset <daemonset-name> --revision=1` | Display the details of a specific revision |
| `kubectl rollout undo daemonset <daemonset-name> --to-revision=<revision>` | Roll back to a specific revision |
| `kubectl rollout status ds/<daemonset-name>` | View the progress of the rollback |
| `kubectl delete daemonset <daemonset_name>` | Delete a daemonset |
