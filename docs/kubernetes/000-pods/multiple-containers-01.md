# Pod with multiple Containers

Pods are designed to support multiple cooperating processes (as containers) that form a cohesive unit of service.

**The containers in a Pod are automatically co-located and co-scheduled on the same physical or virtual machine in the cluster**.

## Example

To create a Pod with multiple containers:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx-cms-daq-workshop-multiple
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 8080
  - name: alpine-cms-daq-workshop
    image: alpine
    args: ["/bin/sh", "-c", "sleep infinity"]
EOF
```

## List Pods

List the Pods and you should notice that a new Pod has been created and running:

```diff
$ kubectl get pods

  NAME                              READY   STATUS    RESTARTS   AGE
  nginx-cms-daq-workshop            1/1     Running   0          20h
+ nginx-cms-daq-workshop-multiple   2/2     Running   0          9s
```

## Describe Pod

We can get details the details of the new Pod by using `kubectl describe` command:

```bash
$ kubectl describe pod nginx-cms-daq-workshop-multiple

Name:         nginx-cms-daq-workshop-multiple
Namespace:    default
Priority:     0
Node:         cms-daq-workshop-gml7jxg5oxyf-node-0/188.185.124.201
Start Time:   Fri, 06 Jan 2023 11:58:08 +0100
Labels:       <none>
Annotations:  cni.projectcalico.org/podIP: 10.100.155.140/32
              cni.projectcalico.org/podIPs: 10.100.155.140/32
Status:       Running
IP:           10.100.155.140
IPs:
  IP:  10.100.155.140
Containers:
  nginx:
    Container ID:   containerd://8fd4aee478e670e8d4755a19fda22f7e3ded46a2b703c98b2e24c3ff58fa9844
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:0047b729188a15da49380d9506d65959cce6d40291ccfb4e039f5dc7efd33286
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Fri, 06 Jan 2023 11:58:10 +0100
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-vjpbt (ro)
  alpine-cms-daq-workshop:
    Container ID:  containerd://3c3985d80ddfc0c331fe850775181cbae65dff571bf92d3ea86f947e7cfcfa8c
    Image:         alpine
    Image ID:      docker.io/library/alpine@sha256:8914eb54f968791faf6a8638949e480fef81e697984fba772b3976835194c6d4
    Port:          <none>
    Host Port:     <none>
    Args:
      /bin/sh
      -c
      sleep infinity
    State:          Running
      Started:      Fri, 06 Jan 2023 11:58:11 +0100
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-vjpbt (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-vjpbt:
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
  Normal  Scheduled  3m17s  default-scheduler  Successfully assigned default/nginx-cms-daq-workshop-multiple to cms-daq-workshop-gml7jxg5oxyf-node-0
  Normal  Pulling    3m17s  kubelet            Pulling image "nginx"
  Normal  Pulled     3m16s  kubelet            Successfully pulled image "nginx" in 1.066994436s
  Normal  Created    3m16s  kubelet            Created container nginx
  Normal  Started    3m16s  kubelet            Started container nginx
  Normal  Pulling    3m16s  kubelet            Pulling image "alpine"
  Normal  Pulled     3m15s  kubelet            Successfully pulled image "alpine" in 1.063829106s
  Normal  Created    3m15s  kubelet            Created container alpine-cms-daq-workshop
  Normal  Started    3m15s  kubelet            Started container alpine-cms-daq-workshop
```
