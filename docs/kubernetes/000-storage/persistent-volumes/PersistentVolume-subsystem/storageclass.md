# Storage Classes

## Introduction

A StorageClass provides a way for administrators to describe the "classes" of storage they offer. Different classes might map to quality-of-service levels, or to backup policies, or to arbitrary policies determined by the cluster administrators. 

!!! info
    This concept is sometimes called "profiles" in other storage systems.


## The StorageClass Resource

Each StorageClass contains the fields `provisioner`, `parameters`, and `reclaimPolicy`, which are used when a PersistentVolume belonging to the class needs to be dynamically provisioned.

Administrators can specify a default StorageClass only for PVCs that don't request any particular class to bind to.

An example of a StorageClass object:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Retain
allowVolumeExpansion: true
mountOptions:
  - debug
volumeBindingMode: Immediate
```

### Default StorageClass

**When a PVC does not specify a `storageClassName`, the default StorageClass is used**. The cluster **can only have one default StorageClass**. If more than one default StorageClass is accidentally set, the newest default is used when the PVC is dynamically provisioned.

### Provisioner

Each StorageClass has a `provisioner` that determines what volume plugin is used for provisioning PVs. Admins must specify this field.

Kubernetes provides _internal provisioners_, which you can see listed [here](https://kubernetes.io/docs/concepts/storage/storage-classes/). Their names have a `kubernetes.io` prefix and they are shipped by default as part of Kubernetes.

Users can specify and run _external provisioners_ - these are independent programs that follow a Kubernetes-defined [specification](https://github.com/kubernetes/design-proposals-archive/blob/main/storage/volume-provisioning.md).

An example of an external provisioner is the Network File System (NFS), which is not available as an internal provisioner, but offers an external provisioner. In some cases, third-party storage vendors provide their own external provisioners.

### Reclaim Policy

PersistentVolumes that are dynamically created by a StorageClass will have the reclaim policy specified in the `reclaimPolicy` field of the class, which can be either `Delete` or `Retain`.  If no reclaimPolicy is specified when a StorageClass object is created, it will default to `Delete`.

PersistentVolumes that are created manually and managed via a StorageClass will have whatever reclaim policy they were assigned at creation.

### Allow Volume Expansion

PersistentVolumes can be configured to be expandable. This feature when set to true, allows the users to resize the volume by editing the corresponding PVC object.

!!! note
    Only specific volume types support volume expansion. Consult the [official documentation for more details](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#expanding-persistent-volumes-claims).

!!! note
    You can only use the volume expansion feature to grow a Volume, not to shrink it.

### Mount Options

PersistentVolumes that are dynamically created by a StorageClass will have the mount options specified in the `mountOptions` field of the class.

If the volume plugin does not support mount options but mount options are specified, provisioning will fail. Mount options are not validated on either the class or PV. If a mount option is invalid, the PV mount fails.

### Volume Binding Mode

The `volumeBindingMode` field of the StorageClass controls when volume binding and dynamic provisioning should occur. If it is not set, the `Immediate` mode is the default.

The `Immediate` mode indicates that volume binding and dynamic provisioning occurs once the PersistentVolumeClaim is created. A PersistentVolumeClaim is an object that represents a request by a pod for a persistent storage volume. For storage backends that are topology-constrained and not globally accessible from all Nodes in the cluster, **PersistentVolumes will be bound or provisioned without knowledge of the Pod's scheduling requirements**. This _may_ result in **unschedulable Pods**.

Cluster administrators can address issues like this by setting the volume binding mode to `WaitForFirstConsumer`. This mode **delays the binding and provisioning of the PersistentVolume until the creation of a pod using a matching PersistentVolumeClaim**.

PersistentVolumes are selected or provisioned according to the pod's scheduling constraints. These include:

- Resource requirements
- Node selectors
- Pod affinity, anti-affinity, taints, and tolerations


!!! note
    If you choose to use `WaitForFirstConsumer`, do not use `nodeName` in the Pod spec to specify node affinity. If `nodeName` is used in this case, the scheduler will be bypassed and PVC will remain in pending state.

    Instead, you can use node selector for hostname in this case.

### Allowed Topologies

When a cluster operator specifies the `WaitForFirstConsumer` volume binding mode, it is no longer necessary to restrict provisioning to specific topologies in most situations. However, if still required, `allowedTopologies` can be specified.

This example demonstrates how to restrict the topology of provisioned volumes to specific zones and should be used as a replacement for the `zone` and `zones` parameters for the supported plugins:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
volumeBindingMode: WaitForFirstConsumer
allowedTopologies:
- matchLabelExpressions:
  - key: topology.kubernetes.io/zone
    values:
    - us-central-1a
    - us-central-1b
```


## Parameters

Storage Classes have parameters that describe volumes belonging to the storage class. Different parameters may be accepted depending on the `provisioner`. For example, the value `io1`, for the parameter `type`, and the parameter `iopsPerGB` are specific to EBS. When a parameter is omitted, some default is used.

Let's see a few examples below. More examples can be found on the official Kubernetes documentation.

A complete [list of CSI drivers which can be used with Kubernetes can be found here](https://kubernetes-csi.github.io/docs/drivers.html).

### NFS

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: example-nfs
provisioner: example.com/external-nfs
parameters:
  server: nfs-server.example.com
  path: /share
  readOnly: "false"
```

A short description of the above fields:

- `server`: Server is the hostname or IP address of the NFS server.
- `path`: Path that is exported by the NFS server.
- `readOnly`: A flag indicating whether the storage will be mounted as read only (default false).

**Kubernetes doesn't include an internal NFS provisioner**. You need to use an external provisioner to create a StorageClass for NFS. Here are some examples:

- [NFS Ganesha server and external provisioner](https://github.com/kubernetes-sigs/nfs-ganesha-server-and-external-provisioner): an out-of-tree dynamic provisioner (meaning its code is not part of the core Kubernetes code) to quickly & easily deploy shared storage that works almost anywhere.
- [NFS subdir external provisioner](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner): an automatic provisioner that use your _existing and already configured_ NFS server to support dynamic provisioning of Kubernetes PersistentVolumes via PersistentVolumeClaims.

### Local

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

Local volumes do not currently support dynamic provisioning, however a StorageClass should still be created to delay volume binding until Pod scheduling. This is specified by the `WaitForFirstConsumer` volume binding mode.

Delaying volume binding allows the scheduler to consider all of a Pod's scheduling constraints when choosing an appropriate PersistentVolume for a PersistentVolumeClaim.

## Dynamic Provisioning

In the next sections we'll see how StorageClass is coupled with _dynamic volume provisioning_, which allows storage volumes to be created on-demand, without having the cluster administrators to make manual calls to the storage provider.

## Notes on Container Storage Interface

The Kubernetes implementation of the [**Container Storage Interface (CSI)**](https://github.com/container-storage-interface/spec/blob/master/spec.md) has been promoted to GA in the Kubernetes v1.13 release.

The main objective of CSI is to standardize the mechanism for exposing all types of storage systems across every container orchestrator.

### Why CSI?

Although prior to CSI Kubernetes provided a powerful volume plugin system, it was challenging to add support for new volume plugins to Kubernetes: volume plugins were “in-tree” meaning their code was part of the core Kubernetes code and shipped with the core Kubernetes binaries—vendors wanting to add support for their storage system to Kubernetes (or even fix a bug in an existing volume plugin) were forced to align with the Kubernetes release process. In addition, third-party storage code caused reliability and security issues in core Kubernetes binaries and the code was often difficult (and in some cases impossible) for Kubernetes maintainers to test and maintain.

**CSI was developed as a standard for exposing arbitrary block and file storage storage systems to containerized workloads** on container orchestration systems like Kubernetes. With the adoption of the Container Storage Interface, the Kubernetes volume layer becomes truly extensible.

### How to use a CSI volume?

Assuming a CSI storage plugin is already deployed on a Kubernetes cluster, users can use CSI volumes through the familiar Kubernetes storage API objects: `PersistentVolumeClaims`, `PersistentVolumes`, and `StorageClasses`.

### Dynamic Provisioning for or CSI Storage plugins

You can enable automatic creation/deletion of volumes for CSI Storage plugins that support dynamic provisioning by creating a StorageClass pointing to the CSI plugin.

The following StorageClass, for example, enables dynamic creation of “fast-storage” volumes by a CSI volume plugin called “csi-driver.example.com”.

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: fast-storage
provisioner: csi-driver.example.com
parameters:
  type: pd-ssd
  csi.storage.k8s.io/provisioner-secret-name: mysecret
  csi.storage.k8s.io/provisioner-secret-namespace: mynamespace
```

### List of CSI Drivers

CSI drivers are developed and maintained by third parties. You can find a non-definitive list of CSI drivers [here](https://kubernetes-csi.github.io/docs/drivers.html).

### Building your own CSI Driver for Kubernetes

CSI drivers in Kubernetes are typically deployed with controller and per-node components.

More info on building your own CSI driver can be found on the [official documentation](https://kubernetes-csi.github.io/docs/developing.html) and in [this high-level overview blog post](https://bluexp.netapp.com/blog/cvo-blg-kubernetes-csi-basics-of-csi-volumes-and-how-to-build-a-csi-driver).

!!! note
    CSI drivers may not be compatible across all Kubernetes releases. Please check the specific CSI driver's documentation for supported deployments steps for each Kubernetes release and a compatibility matrix.

More info about CSI can be found on the [official Kubernetes documentation](https://kubernetes.io/docs/concepts/storage/volumes/#csi).
