# Init Containers

## Example

This example defines a simple Pod that has two init containers. The first waits for myservice, and the second waits for mydb. Once both init containers complete, the Pod runs the app container from its spec section.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app.kubernetes.io/name: MyApp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```


Save the above codeblock to a file named `myapp.yaml`.

You can start this Pod by running:

```bash
kubectl apply -f myapp.yaml
```

And check on its status with:

```bash
$ kubectl get pods myapp-pod

NAME        READY   STATUS     RESTARTS   AGE
myapp-pod   0/1     Init:0/2   0          2m45s
```

For more details you can run:

```bash
kubectl describe pod myapp-pod
```

The output is similar to this:

```bash
Name:          myapp-pod
Namespace:     default
[...]
Labels:        app.kubernetes.io/name=MyApp
Status:        Pending
[...]
Init Containers:
  init-myservice:
[...]
    State:         Running
[...]
  init-mydb:
[...]
    State:         Waiting
      Reason:      PodInitializing
    Ready:         False
[...]
Containers:
  myapp-container:
[...]
    State:         Waiting
      Reason:      PodInitializing
    Ready:         False
[...]
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  4m5s  default-scheduler  Successfully assigned default/myapp-pod to cms-daq-workshop-gml7jxg5oxyf-node-2
  Normal  Pulling    4m5s  kubelet            Pulling image "busybox:1.28"
  Normal  Pulled     4m2s  kubelet            Successfully pulled image "busybox:1.28" in 2.976530827s
  Normal  Created    4m2s  kubelet            Created container init-myservice
  Normal  Started    4m2s  kubelet            Started container init-myservice
```

To see logs for the init containers in this Pod, run:

```bash
$ kubectl logs myapp-pod -c init-myservice # Inspect the first init container

$ kubectl logs myapp-pod -c init-mydb      # Inspect the second init container
```

At this point, those init containers will be waiting to discover Services named `mydb` and `myservice`.

Here's a configuration you can use to make those Services appear:

```bash
---
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
---
apiVersion: v1
kind: Service
metadata:
  name: mydb
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9377
```

Save the above codeblock to a file named `services.yaml`.

To create the `mydb` and `myservice` services:

```bash
$ kubectl apply -f services.yaml

service/myservice created
service/mydb created
```

You'll then see that those init containers complete, and that the `myapp-pod` Pod moves into the Running state:

```bash
$ kubectl get pods myapp-pod

NAME        READY     STATUS    RESTARTS   AGE
myapp-pod   1/1       Running   0          9m
```

### Cleanup

To cleanup the created resources, we'll use the `kubectl delete` command:

```bash
$ kubectl delete -f myapp.yaml

$ kubectl delete -f services.yaml
```

This should delete the Pod and the Services objects created in this page.
