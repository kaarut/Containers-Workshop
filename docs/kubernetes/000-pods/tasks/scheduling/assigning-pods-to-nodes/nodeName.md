
# nodeName

!!! warning
    `nodeName` should only be used with caution and only in specific use-cases.

You can also schedule a pod to one specific node via setting `nodeName`.

`nodeName` is a more direct form of node selection than affinity or `nodeSelector`. `nodeName` is a field in the Pod spec. If the `nodeName` field is not empty, the scheduler ignores the Pod and the kubelet on the named node tries to place the Pod on that node.

Some of the limitations of using `nodeName` to select nodes are:

- If the named node does not exist, the Pod will not run, and in some cases may be automatically deleted.
- If the named node does not have the resources to accommodate the Pod, the Pod will fail and its reason will indicate why, for example OutOfmemory or OutOfcpu.
- Node names are not always predictable or stable.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-nodename
spec:
  nodeName: cms-daq-workshop-gml7jxg5oxyf-node-0 # schedule pod to specific node
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
```

The above Pod manifest creates a pod that will get scheduled on `cms-daq-workshop-gml7jxg5oxyf-node-0` only.

Having said that, `nodeName` should only be used with caution and only in specific use-cases.
