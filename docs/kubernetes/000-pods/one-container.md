# Pod with one Container

## Pod Creation

To create a Pod with a single container:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx-cms-daq-workshop
spec:
  containers:
  - name: nginx-cms-daq-workshop
    image: nginx
    ports:
    - containerPort: 8080
EOF
```

The above is an example of a Pod which consists of a container running the image `nginx`.


## List Pods

We can list Pods using `kubectl get`:

```bash
$ kubectl get pods

NAME                     READY   STATUS    RESTARTS   AGE
nginx-cms-daq-workshop   1/1     Running   0          18s
```

!!! tip
    The `--watch` or `-w` flag will watch for changes to the requested object(s).

!!! info
    By default `kubectl get` lists only the objects in the namespace of the current context.

    The `--all-namespaces` or `-A` flag will list the requested object(s) across all namespaces, while the `--namespace` or `-n` flag will set the namespace for the request.


## Describe Pods

We can get details of a specific resource (or group of resources) by using the `kubectl describe` command:

```bash
$ kubectl describe pod nginx-cms-daq-workshop

Name:         nginx-cms-daq-workshop
Namespace:    default
Priority:     0
Node:         cms-daq-workshop-gml7jxg5oxyf-node-0/188.185.124.201
Start Time:   Thu, 05 Jan 2023 15:43:36 +0100
Labels:       <none>
Annotations:  cni.projectcalico.org/podIP: 10.100.155.139/32
              cni.projectcalico.org/podIPs: 10.100.155.139/32
Status:       Running
IP:           10.100.155.139
IPs:
  IP:  10.100.155.139
Containers:
  nginx-cms-daq-workshop:
    Container ID:   containerd://3b9126769245a093ef5f4cedf3dd6f980c1e5a5caee8aff512844a8b83ec077b
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:0047b729188a15da49380d9506d65959cce6d40291ccfb4e039f5dc7efd33286
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 05 Jan 2023 15:43:44 +0100
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-qgswc (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-qgswc:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  8m6s   default-scheduler  Successfully assigned default/nginx-cms-daq-workshop to cms-daq-workshop-gml7jxg5oxyf-node-0
  Normal  Pulling    8m6s   kubelet            Pulling image "nginx"
  Normal  Pulled     8m     kubelet            Successfully pulled image "nginx" in 6.167118392s
  Normal  Created    7m59s  kubelet            Created container nginx-cms-daq-workshop
  Normal  Started    7m59s  kubelet            Started container nginx-cms-daq-workshop
```

As you can see, the container `nginx-cms-daq-workshop` from the `nginx-cms-daq-workshop` Pod is created and started successfully.

!!! info
    The `kubectl describe` command can be quite useful for debugging purposes.

!!! info
    By default `kubectl describe` shows only details about objects in the namespace of the current context.

    The `--all-namespaces` or `-A` flag will list the requested object(s) across all namespaces, while the `--namespace` or `-n` flag will set the namespace for the request.
