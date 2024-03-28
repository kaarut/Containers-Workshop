# OpenStack @ IT - Block Storage

[OpenStack Cinder](https://clouddocs.web.cern.ch/details/volumes.html) offers block storage with volumes that can be dynamically provisioned and attached to your Kubernetes pods. The component that manages storage provided by Cinder in Kubernetes is the [openstack-cinder-csi](https://github.com/kubernetes/cloud-provider-openstack) CSI driver.

For more info, please consult [IT's OpenStack Kubernetes documentation](https://kubernetes.docs.cern.ch/docs/storage/block/).

## Prerequisites

This guide assumes that openstack-cinder-csi is deployed in your cluster. If not, the `cinder_csi_enabled=True` label needs to be passed in when creating the Kubernetes cluster.

Your cluster comes with a default set of Cinder storage classes. Their names are in the format of `cinder-<VOLUME TYPE>` (reclaim policy set to `Retain`) and `cinder-<VOLUME TYPE>-delete` (reclaim policy set to `Delete`). Please make sure you have enough quota for the volume type you have chosen before creating a PVC in it.

## Dynamically Creating Block Devices

To create a Cinder block device, create a PVC with an appropriate Cinder storage class. The created block device will be formatted with `ext4` by default. To change formatting to some other filesystem, set `fsType: <FILESYSTEM NAME>` in storage class parameters.

Steps:

- Create a PersistentVolumeClaim object with an appropriate Cinder storage class.
- Create a Pod object (or any other workload object that fits your needs) and reference the PersistentVolumeClaim name.

```yaml
# This example creates 1GiB PVC with cinder-standard storage class.
# The PVC is then mounted inside nginx Pod.

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-cinder-vol
spec:
  storageClassName: cinder-standard
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
    ports:
    - containerPort: 80
      protocol: TCP
    volumeMounts:
      - mountPath: /var/lib/www/html
        name: my-vol
  volumes:
  - name: my-vol
    persistentVolumeClaim:
      claimName: my-cinder-vol
      readOnly: false
```

Get all `PersistentVolumeClaim` objects (remember that PVCs are namespaced resources):

```bash
$ kubectl get pvc

NAME                     STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS             AGE
my-cinder-vol            Bound    pvc-e5835d22-5f93-41b3-ab02-14e03d6d910d   1Gi        RWO            cinder-standard   6s
```

A PVC object has been created (which is the object we, as the users, defined in our YAML files).


Now, let's fetch all the PersistentVolume objects (remember that PVs are a cluster-wide resource):

```bash
$ kubectl get pv

NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                            STORAGECLASS             REASON   AGE
pvc-e5835d22-5f93-41b3-ab02-14e03d6d910d   1Gi        RWO            Retain           Bound       default/my-cinder-vol            cinder-standard            18s
```

!!! info
    You may wonder what the word `default` means in the claim name. This is the namespace in which the PersistentVolumeClaim object is located. Namespaces allow objects to be organized into disjoint sets.

As you can see, a new PV object has been created with the requested capacity and its name is the volume ID.
We didn't have to create the PV, it was _dynamically_ created, since we specified the `storageClassName` field in our PersistentVolumeClaim object.

```bash
$ kubectl get pods

NAME                   READY   STATUS    RESTARTS   AGE
nginx                  1/1     Running   0          21s
```

Let's view more information about this Pod:

```bash
$ kubectl describe pod nginx

Name:             nginx
Namespace:        default
<OMITTED OUTPUT..>
Node:             workshop-thrizopo-6et46tqotyid-node-2/188.185.24.123
Containers:
  nginx:
    <OMITTED OUTPUT..>
    Image:          nginx
    State:          Running
    Ready:          True
    Mounts:
      /var/lib/www/html from my-vol (rw)
<OMITTED OUTPUT..>
Volumes:
  my-vol:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  my-cinder-vol
    ReadOnly:   false
<OMITTED OUTPUT..>

Events:
  Type     Reason                  Age   From                     Message
  ----     ------                  ----  ----                     -------
  Warning  FailedScheduling        42s   default-scheduler        0/4 nodes are available: 4 pod has unbound immediate PersistentVolumeClaims. preemption: 0/4 nodes are available: 4 Preemption is not helpful for scheduling.
  Normal   Scheduled               37s   default-scheduler        Successfully assigned default/nginx to workshop-thrizopo-6et46tqotyid-node-2
  Normal   SuccessfulAttachVolume  33s   attachdetach-controller  AttachVolume.Attach succeeded for volume "pvc-e5835d22-5f93-41b3-ab02-14e03d6d910d"
  Normal   Pulled                  30s   kubelet                  Container image "nginx" already present on machine
  Normal   Created                 30s   kubelet                  Created container nginx
  Normal   Started                 30s   kubelet                  Started container nginx
```

The persistent volume has been successfully mounted to our Pod and it's ready to be used.

On the worker node that this Pod is running on, we can see that the Cinder volume has been mounted (SSH to the corresponding worker node and then run the following command):

```diff
$ lsblk -a


  NAME MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
  vda  252:0    0   20G  0 disk
  ├─vda1
  │      252:1    0    1M  0 part
  ├─vda2
  │      252:2    0  127M  0 part
  ├─vda3
  │      252:3    0  384M  0 part /boot
  └─vda4
       252:4    0 19.5G  0 part /var/lib/kubelet/pods/8c09cd60-1092-4328-b6e3-d7c7a58d849f/volume-subpaths/cvmfs-csi-default-local/nodeplugin/7
                                /var/lib/containers/storage/overlay
                                /var
                                /sysroot/ostree/deploy/fedora-coreos/var
                                /usr
                                /etc
                                /
                                /sysroot
+ vdb    252:16   0    1G  0 disk /var/lib/kubelet/pods/3b5fc2ec-3773-4a8b-850a-c8f0702486e5/volumes/kubernetes.io~csi/pvc-e5835d22-5f93-41b3-ab02-14e03d6d910d/mount
+                                 /var/lib/kubelet/plugins/kubernetes.io/csi/cinder.csi.openstack.org/d473f77e19e2ca0605702d1baf6a4dbbcac6942a4f059ed6bdcab2468a4b2b71/globalmount
```


## Releasing and re-using persistent volumes

When you no longer need your application and you delete the pod, the persistent volume is unmounted, but remains bound to the persistent volume claim. If you reference this claim in another pod, the pod gets access to the same volume and its files. For as long as the claim exists, the files in the volume are persisted.

When you no longer need the files or the volume, you simply delete the claim. You might wonder if you will be able to recreate the claim and access the same volume. Let’s find out.

### Releasing a persistent volume

Let’s delete the pod and the claim and see what happens:

```bash
$ kubectl delete pod nginx

pod "nginx" deleted

$ kubectl delete pvc my-cinder-vol

persistentvolumeclaim "my-cinder-vol" deleted
```

Now check the status of the persistent volume:

```bash
$ kubectl get pv

NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                            STORAGECLASS      REASON   AGE
pvc-e5835d22-5f93-41b3-ab02-14e03d6d910d   1Gi        RWO            Retain           Released    default/my-cinder-vol            cinder-standard            8m
```

The `STATUS` column shows the volume as `Released` rather than `Available`, as before the claim was created. The `CLAIM` column still shows the `my-cinder-vol` claim to which it was previously bound, even if the claim no longer exists.

### Attempting to bind to a released persistent volume

What happens if you create the claim again? Is the persistent volume bound to the claim so that it can be reused in a pod?

With the `Retain` reclaim policy, when the PersistentVolumeClaim is deleted, the PersistentVolume still exists and the volume is considered "released". But it is not yet available for another claim because the previous claimant's data remains on the volume. An administrator can manually reclaim the volume with the following steps:

- Delete the PersistentVolume. The associated storage asset in external infrastructure (such as an OpenStack Cinder or an AWS EBS volume) still exists after the PV is deleted.
- Manually clean up the data on the associated storage asset accordingly.
- Manually delete the associated storage asset.

If you want to reuse the same storage asset, create a new PersistentVolume with the same storage asset definition.

Let's re-apply the PVC object on your cluster and let's see what happens:

```bash
$ kubectl apply -f pvc.yaml

persistentvolumeclaim/my-cinder-vol created
```

And for the PV and PVC objects:

```bash
$ kubectl get pv

NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                            STORAGECLASS      REASON   AGE
pvc-939cdfa5-c611-4dfd-a31d-84f785d13fba   1Gi        RWO            Retain           Bound       default/my-cinder-vol            cinder-standard            20h
pvc-e5835d22-5f93-41b3-ab02-14e03d6d910d   1Gi        RWO            Retain           Released    default/my-cinder-vol            cinder-standard            21h


$ kubectl get pvc

NAME                     STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
my-cinder-vol            Bound    pvc-939cdfa5-c611-4dfd-a31d-84f785d13fba   1Gi        RWO            cinder-standard   20h
```

As you can see, the PersistentVolume named `pvc-e5835d22-5f93-41b3-ab02-14e03d6d910d` was not used (and is still in a `Released` status) and a new PersistentVolume object was dynamically provisioned.

### Deleting a released PersistentVolume

To make the volume available again, you must delete and recreate the PersistentVolume object. But will this cause the data stored in the volume to be lost?

Imagine if you had accidentally deleted the pod and the claim and caused a major loss of service. You need to restore the service as soon as possible, with all data intact. Deleting the PersistentVolume object sounds like the last thing you should do but is actually safe. Deleting the object is equivalent to deleting a data pointer. The PersistentVolume object only points to a Cinder volume where the data is actually stored. If you delete and recreate the object, you end up with a new pointer to the same data. Let’s try this.

```bash
$ kubectl get pv

NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                            STORAGECLASS      REASON   AGE
pvc-939cdfa5-c611-4dfd-a31d-84f785d13fba   1Gi        RWO            Retain           Bound       default/my-cinder-vol            cinder-standard            21h
pvc-e5835d22-5f93-41b3-ab02-14e03d6d910d   1Gi        RWO            Retain           Released    default/my-cinder-vol            cinder-standard            22h

$ kubectl delete pv pvc-e5835d22-5f93-41b3-ab02-14e03d6d910d
persistentvolume "pvc-e5835d22-5f93-41b3-ab02-14e03d6d910d" deleted
```

Let's confirm that the Cinder volume has not been deleted. From your browser, navigate to the [OpenStack UI for volumes](https://openstack.cern.ch/project/volumes/). The Cinder volume should be still there, holding all your data.

### Using existing block devices

Existing Cinder volume may be imported into Kubernetes by creating its respective PV and PVC objects.

An example can be found [here](https://kubernetes.docs.cern.ch/docs/storage/block/#using-existing-block-devices).


## Expanding volumes

**Volume expansion** is a stable feature since Kubernetes 1.24.

This feature allows Kubernetes users to simply edit their PersistentVolumeClaim objects and specify new size in PVC Spec and Kubernetes will automatically expand the volume using storage backend and also expand the underlying file system in-use by the Pod without requiring any downtime at all if possible.

A new PersistentVolume is never created to satisfy the claim. Instead, an existing volume is resized.

**Not every volume type however is expandable by default**. Some volume types such as - intree hostpath volumes are not expandable at all. For CSI volumes - the CSI driver must have capability `EXPAND_VOLUME` in controller or node service (or both if appropriate).

openstack-cinder-csi can expand Cinder volumes (raw or formatted with a filesystem) to a larger size. The PVC object needs to be patched with the desired size, which in turn triggers the expansion.

!!! danger "Warning"
    Directly editing the size of a PersistentVolume can prevent an automatic resize of that volume. If you edit the capacity of a PersistentVolume, and then edit the `.spec` of a matching PersistentVolumeClaim to make the size of the PersistentVolumeClaim match the PersistentVolume, then no storage resize happens. The Kubernetes control plane will see that the desired state of both resources matches, conclude that the backing volume size has been manually increased and that no resize is necessary.

Let's re-create (in case it was deleted in any previous step) the PVC and the Pod objects to our cluster:

```bash
$ kubectl apply -f pvc-and-pod.yaml

persistentvolumeclaim/my-cinder-vol unchanged
pod/nginx created
```

Before the volume expansion, let's list the OpenStack volumes once more:

```bash
$ openstack volume list --fit-width

+--------------------------------------+------------------------------------------+-----------+------+----------------------------------------------------------------+
| ID                                   | Name                                     | Status    | Size | Attached to                                                    |
+--------------------------------------+------------------------------------------+-----------+------+----------------------------------------------------------------+
| d4df2715-7f9e-4543-b1e1-38939ffea550 | pvc-939cdfa5-c611-4dfd-a31d-84f785d13fba | in-use    |    1 | Attached to workshop-thrizopo-6et46tqotyid-node-2 on /dev/vdb  |
| 3293ab81-a9f9-494a-9447-6c2cabc4e07e | pvc-e5835d22-5f93-41b3-ab02-14e03d6d910d | available |    1 |                                                                |
+--------------------------------------+------------------------------------------+-----------+------+----------------------------------------------------------------+
```

Now, let's edit or patch an existing Cinder PersistentVolumeClaim object by setting the desired value in `.spec.resources.requests.storage` in [Quantity](https://kubernetes.io/docs/reference/kubernetes-api/common-definitions/quantity/#Quantity) units:

```bash
$ # This command patches the PVC to the new size, from 1 to 2 GiB.
$ kubectl patch pvc my-cinder-vol -p '{"spec": {"resources": {"requests": {"storage": "2Gi"}}}}'
persistentvolumeclaim/my-cinder-vol patched
```

After the expansion, let's list the OpenStack volumes:

```bash
$ openstack volume list --fit-width

+--------------------------------------+------------------------------------------+-----------+------+----------------------------------------------------------------+
| ID                                   | Name                                     | Status    | Size | Attached to                                                    |
+--------------------------------------+------------------------------------------+-----------+------+----------------------------------------------------------------+
| d4df2715-7f9e-4543-b1e1-38939ffea550 | pvc-939cdfa5-c611-4dfd-a31d-84f785d13fba | in-use    |    2 | Attached to workshop-thrizopo-6et46tqotyid-node-2 on /dev/vdb  |
| 3293ab81-a9f9-494a-9447-6c2cabc4e07e | pvc-e5835d22-5f93-41b3-ab02-14e03d6d910d | available |    1 |                                                                |
+--------------------------------------+------------------------------------------+-----------+------+----------------------------------------------------------------+
```

and the PVC objects:

```bash
$ kubectl get pvc

NAME                     STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
my-cinder-vol            Bound    pvc-939cdfa5-c611-4dfd-a31d-84f785d13fba   2Gi        RWO            cinder-standard   24h
```

!!! note
    It might take a couple of minutes for the new value to be updated on the corresponding PVC object.

As a last step, let's verify that this has also been propagated to the container using the [`lsblk`](https://man7.org/linux/man-pages/man8/lsblk.8.html) command:

```bash
$ kubectl exec -it nginx -- sh -c 'lsblk -a'

NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
vda    252:0    0   20G  0 disk
|-vda1 252:1    0    1M  0 part
|-vda2 252:2    0  127M  0 part
|-vda3 252:3    0  384M  0 part
`-vda4 252:4    0 19.5G  0 part /etc/resolv.conf
                                /etc/hostname
                                /dev/termination-log
                                /etc/hosts
vdb    252:16   0    2G  0 disk /var/lib/www/html
```

## Snapshotting and creating volumes from snapshots

In Kubernetes, **a `VolumeSnapshot` represents a snapshot of a volume on a storage system**.

openstack-cinder-csi can create snapshots of Cinder volumes, and create Cinder volumes from snapshots.

An example can be found [here](https://kubernetes.docs.cern.ch/docs/storage/block/#snapshotting-and-creating-volumes-from-snapshots).
