# nodeSelector

`nodeSelector` is the simplest recommended form of node selection constraint. You can add the `nodeSelector` field to your Pod specification and specify the node labels you want the target node to have. Kubernetes only schedules the Pod onto nodes that have each of the labels you specify.

For example, the following Pod manifest describes a Pod that has a node selector, `disktype: ssd`. This means that the pod will get scheduled on a node that has a `disktype=ssd label`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-nodeselector
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    disktype: ssd
```

Verify that the pod is running on your chosen node:

```bash
kubectl get pod nginx-nodeselector --output=wide
```

The output is similar to this (assuming that the `cms-daq-workshop-gml7jxg5oxyf-node-0` worker node already has the `disktype=ssd` label):

```bash
NAME                 READY   STATUS    RESTARTS   AGE   IP               NODE                                   NOMINATED NODE   READINESS GATES
nginx-nodeselector   1/1     Running   0          32m   10.100.155.138   cms-daq-workshop-gml7jxg5oxyf-node-0   <none>           <none>
```

!!! note
    If you have specified in the `nodeSelector` field node labels that don't match any node, the Pod will not be scheduled and remain in `Pending` status.
