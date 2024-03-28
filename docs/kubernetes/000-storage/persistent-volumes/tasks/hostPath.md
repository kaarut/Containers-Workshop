# Configure a Pod to Use a hostPath PersistentVolume for Storage

This page shows you how to configure a Pod to use a PersistentVolumeClaim for storage. Here is a summary of the process:

1. You, as cluster administrator, create a PersistentVolume backed by physical storage. You do not associate the volume with any Pod.
1. You, now taking the role of a developer / cluster user, create a PersistentVolumeClaim that is automatically bound to a suitable PersistentVolume.
1. You create a Pod that uses the above PersistentVolumeClaim for storage.

## Create an index.html file on your Node

SSH in one of your worker nodes, create a directory under `/mnt/data` and then, in the `/mnt/data` directory, create an `index.html` file (note: make sure to replace the `IP_ADDRESS` with your worker node's IP address):

```bash
ssh core@IP_ADDRESS
```

Once SSHed in the worker node, create the directory and the file:

```bash
$ sudo su

$ mkdir -p /mnt/data && echo 'Hello from Kubernetes storage' > /mnt/data/index.html
```

Test that the `index.html` file exists:

```bash
cat /mnt/data/index.html
```

## Create a PersistentVolume

In this exercise, you create a _`hostPath`_ PersistentVolume. Kubernetes supports hostPath for development and testing on a single-node cluster. A hostPath PersistentVolume uses a file or directory on the Node to emulate network-attached storage.

!!! warning
    In a production cluster, you would not use `hostPath`. Instead a cluster administrator would provision a network resource like an OpenStack Block Storage volume (Cinder), an NFS share, or an Amazon Elastic Block Store volume. Cluster administrators can also use [StorageClasses](./../PersistentVolume-subsystem/storageclass.md) to set up _dynamic provisioning_.

The configuration file for the `hostPath` PersistentVolume:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-hostpath-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

The configuration file specifies:
- the volume is at `/mnt/data` on the cluster's Node
- a size of 2 gibibytes
- an access mode of `ReadWriteOnce`, which means the volume can be mounted as read-write by a single Node
- the StorageClass name `manual` for the PersistentVolume, which will be used to bind PersistentVolumeClaim requests to this PersistentVolume

Apply the above PV object in your cluster and then view information about it:

```bash
kubectl get pv task-hostpath-pv-volume
```

## Create a PersistentVolumeClaim

The next step is to create a PersistentVolumeClaim. **Pods use PersistentVolumeClaims to request physical storage**. In this exercise, you create a PersistentVolumeClaim that requests a volume of at least two gibibytes that can provide read-write access for at most one Node at a time.

The configuration file for the PersistentVolumeClaim:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-hostpath-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

After you create the PersistentVolumeClaim, **the Kubernetes control plane looks for a PersistentVolume that satisfies the claim's requirements**. If the control plane finds a suitable PersistentVolume with the same StorageClass, it binds the claim to the volume.

Look again at the PersistentVolume::

```bash
kubectl get pv task-hostpath-pv-volume
```

Now the output shows a `STATUS` of `Bound`:

```bash
NAME             CAPACITY   ACCESSMODES   RECLAIMPOLICY   STATUS    CLAIM                   STORAGECLASS   REASON    AGE
task-hostpath-pv-volume   5Gi        RWO            Retain           Bound    default/task-hostpath-pv-claim   manual                  21s
```

Let's have a look at the PersistentVolumeClaim:

```bash
kubectl get pvc task-hostpath-pv-claim
```

The output shows that the PersistentVolumeClaim is bound to your PersistentVolume, `task-hostpath-pv-volume`:

```bash
NAME            STATUS    VOLUME           CAPACITY   ACCESSMODES   STORAGECLASS   AGE
task-hostpath-pv-claim   Bound    task-hostpath-pv-volume   5Gi        RWO            manual         81s
```

## Create a Pod

The next step is to create a Pod that uses your PersistentVolumeClaim as a volume.

The configuration file for the Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: task-hostpath-pv-pod
spec:
  volumes:
  - name: task-hostpath-pv-storage
    persistentVolumeClaim:
      claimName: task-hostpath-pv-claim
  containers:
  - name: task-hostpath-pv-container
    image: nginx
    ports:
    - containerPort: 80
      name: "http-server"
    volumeMounts:
    - mountPath: "/usr/share/nginx/html"
      name: task-hostpath-pv-storage
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/hostname
            operator: In
            values:
            - workshop-thrizopo-6et46tqotyid-node-0
```

!!! note
    In the above Pod object, we only specify the node affinity rule so that the Pod is being scheduled on the worker node we created the `index.html` file. Make sure to modify this value accordingly.

Notice that the Pod's configuration file specifies a PersistentVolumeClaim, but it does not specify a PersistentVolume. From the Pod's point of view, the claim is a volume.

Apply the Pod object to your Kubernetes cluster.

Then, get a shell to the container running in your Pod:

```bash
kubectl exec -it task-hostpath-pv-pod -- /bin/bash
```

In your shell, verify that nginx is serving the `index.html` file from the hostPath volume:

```bash
# Be sure to run these 3 commands inside the root shell that comes from
# running "kubectl exec" in the previous step
apt update
apt install curl
curl http://localhost/
```

The output shows the text that you wrote to the `index.html` file on the `hostPath` volume:

```bash
Hello from Kubernetes storage
```

If you see that message, you have successfully configured a Pod to use storage from a PersistentVolumeClaim.
