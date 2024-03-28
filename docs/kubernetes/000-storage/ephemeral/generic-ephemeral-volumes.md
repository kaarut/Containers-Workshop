# Generic ephemeral volumes

Generic ephemeral volumes are similar to `emptyDir` volumes in the sense that they provide a per-pod directory for scratch data that is usually empty after provisioning. But they may also have additional features:

- Storage can be local or network-attached.
- Volumes can have a fixed size that Pods are not able to exceed.
- Volumes may have some initial data, depending on the driver and parameters.
- Typical operations on volumes are supported assuming that the driver supports them, including [snapshotting](https://kubernetes.io/docs/concepts/storage/volume-snapshots/), [cloning](https://kubernetes.io/docs/concepts/storage/volume-pvc-datasource/), [resizing](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#expanding-persistent-volumes-claims), and [storage capacity tracking](https://kubernetes.io/docs/concepts/storage/storage-capacity/).

!!! note
    This feature is in stable state since Kubernetes v1.23.


## Example

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: generic-ephemeral-volume
spec:
  containers:
    - name: my-frontend
      image: busybox:1.28
      volumeMounts:
      - mountPath: "/scratch"
        name: scratch-volume
      command: [ "sleep", "1000000" ]
  volumes:
    - name: scratch-volume
      ephemeral:
        volumeClaimTemplate:
          metadata:
            labels:
              type: my-frontend-volume
          spec:
            accessModes: [ "ReadWriteOnce" ]
            storageClassName: "scratch-storage-class"
            resources:
              requests:
                storage: 1Gi
```
