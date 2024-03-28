
# Local PersistentVolume

A **`local` volume represents a mounted local storage device such as a disk**, partition or directory.

Local volumes can **_only_ be used as a statically created PersistentVolume**. Dynamic provisioning is _not_ supported.

## Local vs hostPath volumes

In a previous chapter, you learned that you can use a `hostPath` volume in a pod if you want the pod to access part of the host’s filesystem. Now you’ll learn how to do the same with PersistentVolumes. You might wonder why I need to teach you another way to do the same thing, but it’s really not the same.

Compared to `hostPath` volumes, `local` volumes are used in a durable and portable manner without manually scheduling pods to nodes. The system is aware of the volume's node constraints by looking at the node affinity on the PersistentVolume.

HostPath volumes mount a file or directory from the host node’s filesystem into a Pod. Similarly a Local Persistent Volume mounts a local disk or partition into a Pod.

The biggest difference is that **the Kubernetes scheduler understands which node a `local` PersistentVolume belongs to**. With hostPath volumes, a pod referencing a `hostPath` volume may be moved by the scheduler to a different node resulting in data loss. But with `local` PersistentVolumes, the Kubernetes scheduler ensures that a pod using a `local` PersistentVolume is always scheduled to the same node.

`local` volumes are subject to the availability of the underlying node and are not suitable for all applications. **If a node becomes unhealthy**, then the **local volume becomes inaccessible** by the pod. The **pod using this volume is unable to run**. Applications **using `local` volumes must be able to tolerate this reduced availability**, as well as potential data loss, depending on the durability characteristics of the underlying disk.

While `hostPath` volumes may be referenced via a PersistentVolumeClaim (PVC) or directly inline in a pod definition, `local` PersistentVolumes can only be referenced via a PVC.

!!! note
    Local PersistentVolumes are also better than `hostPath` volumes because they offer much better security. As explained in a previous chapter, you don’t want to allow regular users to use `hostPath` volumes at all. Because persistent volumes are managed by the cluster administrator, regular users can’t use them to access arbitrary paths on the host node.


## Example

The following example shows a PersistentVolume using a `local` volume and `nodeAffinity`.

### StorageClass creation

Local volumes do not currently support dynamic provisioning, however a StorageClass should still be created to delay volume binding until Pod scheduling. This is specified by the `WaitForFirstConsumer` volume binding mode:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

Delaying volume binding allows the scheduler to consider all of a Pod's scheduling constraints when choosing an appropriate PersistentVolume for a PersistentVolumeClaim.

### PersistentVolume creation

After the StorageClass creation for local storage, the external static provisioner can be configured and run to create PVs for all the local disks on your nodes.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv-local
spec:
  capacity:
    storage: 5Gi
  volumeMode: Block
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /mnt/disks/ssd1
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - workshop-thrizopo-6et46tqotyid-node-0
```

You **must set a PersistentVolume `nodeAffinity` when using `local` volumes**. The Kubernetes scheduler uses the PersistentVolume `nodeAffinity` to schedule these Pods to the correct node.

PersistentVolume `.spec.volumeMode` can be set to `Block` (instead of the default value `Filesystem`) to expose the local volume as a raw block device.

Even though we created two of the needed components for `local` type volumes, we need a PersistentVolumeClaim (PVC) and then reference in our pod specification.
