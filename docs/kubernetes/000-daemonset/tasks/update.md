
# DaemonSet Update

This page shows how to perform a rolling update on a DaemonSet.

## Prerequisites

The following prerequisites should be met:

- Have the `fluentd-elasticsearch` DaemonSet object applied to your cluster, as described in the initial chapter about DaemonSets.

## Update Strategy

DaemonSet has two update strategy types:

- `OnDelete`: With `OnDelete` update strategy, after you update a DaemonSet template, new DaemonSet pods will only be created when you manually delete old DaemonSet pods. This is the same behavior of DaemonSet in Kubernetes version 1.5 or before.
- `RollingUpdate`: This is the default update strategy.
    With `RollingUpdate` update strategy, after you update a DaemonSet template, old DaemonSet pods will be killed, and new DaemonSet pods will be created automatically, in a controlled fashion. At most one pod of the DaemonSet will be running on each node during the whole update process.

## Performing a Rolling Update

To enable the rolling update feature of a DaemonSet, you must set its `.spec.updateStrategy.type` to `RollingUpdate`.

You may want to set `.spec.updateStrategy.rollingUpdate.maxUnavailable` (default to 1), `.spec.minReadySeconds` (default to 0) and `.spec.updateStrategy.rollingUpdate.maxSurge` (defaults to 0) as well:

```yaml
spec:
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
```

## Watching the rolling update status 

Update rollouts can be monitored by running the following command:

```bash
kubectl rollout status ds fluentd-elasticsearch -n kube-system
```

## Exercise

Edit the `fluentd-elasticsearch` DaemonSet in the `kube-system` namespace to update the fluentd image to `quay.io/fluentd_elasticsearch/fluentd:v2.6.0` and watch how the rolling update behaves.

!!! tip
    This can be achieved in a few ways:

    - updating the YAML configuration file and then use `kubectl apply`.
    - using `kubectl edit` to edit the DaemonSet on the fly.
    - using the `kubectl set image`:
        ```
        kubectl set image ds/fluentd-elasticsearch \
            fluentd-elasticsearch=quay.io/fluentd_elasticsearch/fluentd:v2.6.0 \
            -n kube-system
        ```
