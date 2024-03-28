# PersistentVolumeClaim

Each PersistentVolumeClaim (PVC) contains a spec and status, which is the specification and status of the claim. The name of a PersistentVolumeClaim object must be a [valid DNS subdomain name](./../../../000-objects/names-and-ids.md).

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 2Gi
  storageClassName: slow
  selector:
    matchLabels:
      release: "stable"
    matchExpressions:
      - {key: environment, operator: In, values: [dev]}
```

!!! tip
    As an application developer, you should never include persistent volume definitions in your application manifests. You should include persistent volume claims because they specify the storage requirements of your application.

## Configuration

### Access Modes

Claims use [the same conventions as volumes](./pv.md#access-modes) when requesting storage with specific access modes.

### Volume Modes

Claims use [the same convention as volumes](./pv.md#volume-mode) to indicate the consumption of the volume as either a filesystem or block device.

### Resources

Claims, like Pods, can request specific quantities of a resource. In this case, the request is for storage. The same resource model applies to both volumes and claims.

### Selector

Claims can specify a [label selector](./../../../000-objects/labels-and-selectors.md#label-selectors) to further filter the set of volumes. Only the volumes whose labels match the selector can be bound to the claim. The selector can consist of two fields:

- `matchLabels` - the volume must have a label with this value.
- `matchExpressions` - a list of requirements made by specifying key, list of values, and operator that relates the key and values. Valid operators include In, NotIn, Exists, and DoesNotExist.

All of the requirements, from both `matchLabels` and `matchExpressions`, are ANDed together – they must all be satisfied in order to match.


### Class

A claim can request a particular class by specifying the name of a StorageClass using the attribute `storageClassName`. Only PVs of the requested class, ones with the same `storageClassName` as the PVC, can be bound to the PVC.

**PVCs don't necessarily have to request a class**. A PVC with its `storageClassName` set equal to `""` is always interpreted to be requesting a PV with no class, so it can only be bound to PVs with no class (no annotation or one set equal to `""`). A PVC with no `storageClassName` is not quite the same and is treated differently by the cluster, depending on whether the [`DefaultStorageClass` admission plugin](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#defaultstorageclass) is turned on.

The field `storageClassName` is used for dynamic provisioning of persistent volumes. It must be set to an empty string if you want Kubernetes to bind the pre-provisioned persistent volume to this claim instead of dynamically provisioning a new persistent volume.

!!! note
    Currently, a PVC with a non-empty `selector` can't have a PV dynamically provisioned for it.
