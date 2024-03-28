# NFS volumes

## Getting started

!!! info
    Network File System (NFS) is a distributed file system protocol, allowing a user on a client computer to access files over a computer network much like local storage is accessed.

The **NFS must already exist** – Kubernetes doesn't run the NFS, pods in just access it.

An `nfs` volume allows an existing NFS share to be mounted into a Pod.

Some benefits of using NFS:

1. what's already stored in the NFS is not deleted when a pod is destroyed. Data is persistent.
1. an NFS can be accessed from multiple pods at the same time. An NFS can be used to share data between pods.


## Example

In the example below, we create a Pod with one container. We add the NFS volume to the Pod and we set its `server` and `path` values to point to the NFS server. Then, mount the NFS volume in the container.


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-using-nfs
spec:
  volumes:
  - name: nfs-volume
    nfs: 
      # URL for the NFS server
      server: 10.108.211.244
      path: /
  containers:
  - name: nginx
    image: nginx
    # Mount the NFS volume in the container
    volumeMounts:
    - name: nfs-volume
      mountPath: /var/nfs
```

If the NFS server cannot be reached and/or mounted, Kubernetes will provide relavant information as Events (the following output is fetched using the `kubectl describe pod` command):

```bash
Events:
  Type     Reason       Age                   From               Message
  ----     ------       ----                  ----               -------
  Normal   Scheduled    6m26s                 default-scheduler  Successfully assigned default/nginx-using-nfs to cms-daq-workshop-gml7jxg5oxyf-node-2
  Warning  FailedMount  2m6s (x2 over 4m23s)  kubelet            Unable to attach or mount volumes: unmounted volumes=[nfs-volume], unattached volumes=[nfs-volume kube-api-access-668jf]: timed out waiting for the condition
  Warning  FailedMount  6s (x3 over 4m20s)    kubelet            MountVolume.SetUp failed for volume "nfs-volume" : mount failed: exit status 32
Mounting command: mount
Mounting arguments: -t nfs 10.108.211.244:/ /var/lib/kubelet/pods/d36f2cfd-22f3-42e0-b8ba-7a0948378379/volumes/kubernetes.io~nfs/nfs-volume
Output: mount.nfs: Connection timed out
```


!!! note
    You can't specify NFS mount options in a Pod spec. You can either set mount options server-side or use [`/etc/nfsmount.conf`](https://man7.org/linux/man-pages/man5/nfsmount.conf.5.html). You can also mount NFS volumes via [PersistentVolumes](./PersistentVolume-subsystem/getting-started.md) which do allow you to set mount options.


## Sharing data between pods

You can use the NFS volume from the example above to share data between pods in your cluster.

Just add the volume to each pod, and add a volume mount to use the NFS volume from each container.

### Caveats

Typically, the underlying storage technology doesn’t allow a volume to be attached to more than one node at a time in read/write mode, but multiple pods on the same node can all use the volume in read/write mode.

For most storage technologies available **in the cloud**, **the only way to use the same network volume on multiple nodes simultaneously is to mount them in read-only mode**.
