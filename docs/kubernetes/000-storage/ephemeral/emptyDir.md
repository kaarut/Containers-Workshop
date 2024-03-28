# emptyDir

A few key points on Kubernetes' `emptyDir` volume:

- An `emptyDir` volume is a **directory that is created when a new Pod is being scheduled/created and resides on the local node’s filesystem** with no contents initially.
- An `emptyDir` volume is first created when a Pod is assigned to a node, and **exists as long as that Pod is running on that node**.
- As the name says, the `emptyDir` volume is **initially empty**. All containers in the Pod can read and write the same files in the `emptyDir` volume, though that volume can be mounted at the same or different paths in each container.
- When a Pod is removed from a node for any reason, the data in the `emptyDir` is deleted permanently.

!!! note
    A container crashing does _not_ remove a Pod from a node. The data in an `emptyDir` volume is safe across container crashes.


## Use-cases

A `hostPath` volume is not a good place to store the data of a database. Because the contents of the volume are stored on the filesystem of a specific node, the database pod will not be able to see the data if it gets rescheduled to another node.

Typically, a hostPath volume is **used in cases where the pod actually needs to read or write files written or read by the node itsel**f, such as system-level logs.

Some other uses for an emptyDir are:

- scratch space, such as for a disk-based merge sort
- checkpointing a long computation for recovery from crashes
- holding files that a content-manager container fetches while a webserver container serves the data
- local cache or a means to share data between different containers of a Pod

## Example

This Pod has a Volume of type `emptyDir` that lasts for the life of the Pod, even if the Container terminates and restarts:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis
    volumeMounts:
    - name: redis-storage
      mountPath: /data/redis
  volumes:
  - name: redis-storage
    emptyDir: {}
```


## Storage Medium

These volumes are stored either on the **node’s backing disk storage** _or_ **memory**.

The `emptyDir.medium` field controls where `emptyDir` volumes are stored. By default `emptyDir` volumes are stored on whatever medium that backs the node such as disk, SSD, or network storage, depending on your environment.

If you set the `emptyDir.medium` field to `"Memory"`, Kubernetes mounts a tmpfs (RAM-backed filesystem) for you instead. While tmpfs is very fast, be aware that unlike disks, tmpfs is cleared on node reboot and any files you write count against your container's memory limit.


## Size Limit

A size limit can be specified for the default medium, which limits the capacity of the `emptyDir` volume.

!!! note
    The storage is allocated from [node ephemeral storage](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#setting-requests-and-limits-for-local-ephemeral-storage). If that is filled up from another source (for example, log files or image overlays), the emptyDir may run out of capacity before this limit.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-sizelimit
spec:
  containers:
  - image: nginx
    name: nginx
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir:
      sizeLimit: 100Mi
```

Let's create a file that exceeds the `sizeLimit` value and save it on the mounted volume:

```bash
kubectl exec -it emptydir-sizelimit -- dd if=/dev/urandom of=/cache/sample.txt bs=1G count=6
```


After a few seconds (sometimes it can take a couple of minutes), the Pod will be evicted and killed. The following events can be viewed with the `kubectl describe` command:

```bash
Events:
  Type     Reason     Age    From               Message
  ----     ------     ----   ----               -------
  Normal   Scheduled  2m22s  default-scheduler  Successfully assigned default/emptydir-sizelimit to cms-daq-workshop-gml7jxg5oxyf-node-2
  Normal   Pulling    2m22s  kubelet            Pulling image "nginx"
  Normal   Pulled     2m21s  kubelet            Successfully pulled image "nginx" in 1.018572802s
  Normal   Created    2m21s  kubelet            Created container nginx
  Normal   Started    2m21s  kubelet            Started container nginx
  Warning  Evicted    10s    kubelet            Usage of EmptyDir volume "cache-volume" exceeds the limit "100Mi".
  Normal   Killing    10s    kubelet            Stopping container nginx
```


## Location oh Host Machine

The location of the `emptyDir` volumes should be in `/var/lib/kubelet/pods/{POD_ID}/volumes/kubernetes.io~empty-dir/` on the given node where your pod is running.

!!! info "Reminder"
    Every Pod in Kubernetes is assigned a unique ID, which can be retrieved with `kubectl get pod <POD_NAME> -o=jsonpath='{.metadata.uid}{"\n"}'`.


## Notes

The storage media (such as Disk or SSD) of an `emptyDir` volume is determined by the medium of the filesystem holding the kubelet root dir (typically `/var/lib/kubelet`). **There is no limit on how much space an `emptyDir` or `hostPath` volume can consume, and no isolation between containers or between pods**.
