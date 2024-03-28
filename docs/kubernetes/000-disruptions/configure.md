# Specifying a Disruption Budget for your Application

This page shows how to limit the number of concurrent disruptions that your application experiences, allowing for higher availability while permitting the cluster administrator to manage the clusters nodes.

The most common use case when you want to protect an application specified by one of the built-in Kubernetes controllers:

- Deployment
- StatefulSet
- etc.

In this case, make a note of the controller's `.spec.selector`; the same selector goes into the PDBs `.spec.selector`.


## Rounding logic when specifying percentages

Values for `minAvailable` or `maxUnavailable` can be expressed as integers or as a percentage: 

- When you specify an integer, it represents a number of Pods. For instance, if you set `minAvailable` to 10, then 10 Pods must always be available, even during a disruption.
- When you specify a percentage by setting the value to a string representation of a percentage (eg. `"50%"`), it represents a percentage of total Pods. For instance, if you set `minAvailable` to `"50%"`, then at least 50% of the Pods remain available during a disruption.

When you specify the value as a percentage, it may not map to an exact number of Pods. For example, if you have 7 Pods and you set minAvailable to "50%", it's not immediately obvious whether that means 3 Pods or 4 Pods must be available. Kubernetes rounds up to the nearest integer, so in this case, 4 Pods must be available.

## Specifying a PodDisruptionBudget

A PodDisruptionBudget has three fields:

- A label selector `.spec.selector` to specify the set of Pods to which it applies. This field is required.
- `.spec.minAvailable`: the number of Pods from that set that must still be available after the eviction, even in the absence of the evicted pod. It can be either an absolute number or a percentage.
- `.spec.maxUnavailable`: the number of Pods from that set that can be unavailable after the eviction. It can be either an absolute number or a percentage.

You can specify only one of `maxUnavailable` and `minAvailable` in a single PodDisruptionBudget. `maxUnavailable` can only be used to control the eviction of pods that have an associated controller managing them.

!!! tip
    The use of `maxUnavailable` is recommended as it automatically responds to changes in the number of replicas of the corresponding controller.


Example PDB Using `maxUnavailable`. They match Pods with the label `app=nginx`.

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: nginx-pdb
spec:
  maxUnavailable: 1
  selector:
    matchLabels:
      app: nginx
```

## Check the status of the PDB

- Once you've created a PDB object, you can check the status of it:

    ```bash
    kubectl get poddisruptionbudgets
    ```

    _or_ use the shortname:

    ```bash
    kubectl get pdb
    ```

- Assuming you don't actually have pods matching `app: nginx` in your namespace, then you'll see something like this:

    ```bash
    NAME     MIN AVAILABLE   MAX UNAVAILABLE   ALLOWED DISRUPTIONS   AGE
    nginx-pdb   N/A             1                 0                     13s
    ```

- If there are matching Pods, then you would see something like this:

    ```bash
    $ kubectl get pdb

    NAME        MIN AVAILABLE   MAX UNAVAILABLE   ALLOWED DISRUPTIONS   AGE
    nginx-pdb   N/A             1                 1                     3s
    ```

    The non-zero value for `ALLOWED DISRUPTIONS` means that the disruption controller has seen the pods, counted the matching pods, and updated the status of the PDB.

- You can get more information about the status of a PDB with this command:

    ```bash
    kubectl get poddisruptionbudgets nginx-pdb -o yaml
    ```

    and the output should look something like this:

    ```yaml
    apiVersion: policy/v1
    kind: PodDisruptionBudget
    metadata:
    annotations:
    name: nginx-pdb
    …
    status:
    currentHealthy: 10
    desiredHealthy: 9
    disruptionsAllowed: 1
    expectedPods: 10
    observedGeneration: 1
    ```


## Notes

- If you set `maxUnavailable` to 0% or 0, or you set `minAvailable` to 100% or the number of replicas, you are requiring zero voluntary evictions. When you set zero voluntary evictions for a workload object such as ReplicaSet, then you cannot successfully drain a Node running one of those Pods. If you try to drain a Node where an unevictable Pod is running, the drain never completes. This is permitted as per the semantics of `PodDisruptionBudget`.
