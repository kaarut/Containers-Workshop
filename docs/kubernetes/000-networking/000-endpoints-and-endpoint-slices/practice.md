# Endpoints and EndpointSlices in action

As we mentioned in the previous section, when Kubernetes processes a Service description, and if the service selector matches a pod label, **Kubernetes will automatically create an Endpoints and EndpointSlice object with the same name as the Service**, which stores the Pod’s IP address and port.

An easy way to investigate and see the relationship is:

- `kubectl describe pods` - to observe the IP addresses of your Pods
- `kubectl get ep` - to observe the IP addresses assigned to your Endpoint
- `kubectl describe service myServiceName` - to observe the Endpoints associated with your Service

## Getting familiar with Endpoints and commands

### Prerequisites

In this page we'll have a closer look on how Services, Endpoints/EndpointSlices and Pods are connected. The following prerequisites should be met:

- The `nginx-deployment` Deployment object, as described [here](../../000-deployments/getting-started.md#creating-a-deployment).
- The `nginx-clusterip` Service object, as described [here](../000-services/clusterIP.md#service-creation).


### Listing resources

- Get the list of all **Pods** matching the `app=nginx` label selector:

    ```bash
    kubectl get pods -l app=nginx -o wide
    ```

    The output should be similar to this one:

    ```bash
    NAME                                READY   STATUS    RESTARTS   AGE   IP               NODE                                   NOMINATED NODE   READINESS GATES
    nginx-deployment-66b6c48dd5-4ckx7   1/1     Running   0          39m   10.100.126.80    cms-daq-workshop-gml7jxg5oxyf-node-1   <none>           <none>
    nginx-deployment-66b6c48dd5-7lqjh   1/1     Running   0          39m   10.100.126.83    cms-daq-workshop-gml7jxg5oxyf-node-1   <none>           <none>
    nginx-deployment-66b6c48dd5-7rqnp   1/1     Running   0          39m   10.100.155.154   cms-daq-workshop-gml7jxg5oxyf-node-0   <none>           <none>
    nginx-deployment-66b6c48dd5-89fsc   1/1     Running   0          39m   10.100.155.149   cms-daq-workshop-gml7jxg5oxyf-node-0   <none>           <none>
    nginx-deployment-66b6c48dd5-89zpj   1/1     Running   0          39m   10.100.126.82    cms-daq-workshop-gml7jxg5oxyf-node-1   <none>           <none>
    nginx-deployment-66b6c48dd5-h9fbx   1/1     Running   0          39m   10.100.155.151   cms-daq-workshop-gml7jxg5oxyf-node-0   <none>           <none>
    nginx-deployment-66b6c48dd5-ph9kw   1/1     Running   0          39m   10.100.126.81    cms-daq-workshop-gml7jxg5oxyf-node-1   <none>           <none>
    nginx-deployment-66b6c48dd5-q865s   1/1     Running   0          39m   10.100.155.153   cms-daq-workshop-gml7jxg5oxyf-node-0   <none>           <none>
    nginx-deployment-66b6c48dd5-v6fck   1/1     Running   0          39m   10.100.155.150   cms-daq-workshop-gml7jxg5oxyf-node-0   <none>           <none>
    nginx-deployment-66b6c48dd5-wn9v6   1/1     Running   0          39m   10.100.126.79    cms-daq-workshop-gml7jxg5oxyf-node-1   <none>           <none>
    ```

    which is the list of all Pods matching a label selector and their IPs.

- View details about the `nginx-clusterip` **Service** (assuming that the prerequisites of this page have been met):

    ```bash
    kubectl describe service nginx-clusterip
    ```

    The output should be similar to this:

    ```diff
      Name:              nginx-clusterip
      Namespace:         default
      Labels:            <none>
      Annotations:       <none>
      Selector:          app=nginx
      Type:              ClusterIP
      IP Family Policy:  SingleStack
      IP Families:       IPv4
      IP:                10.254.201.125
      IPs:               10.254.201.125
      Port:              http  80/TCP
      TargetPort:        80/TCP
    + Endpoints:         10.100.126.79:80,10.100.126.80:80,10.100.126.81:80 + 7 more...
      Session Affinity:  None
      Events:            <none>
    ```

    The Endpoints field should contain all healthy endpoints of the Pods matching the `app=nginx` label.

-  List all **Endpoints**:

    ```bash
    kubectl get endpoints
    ```

    _or_ you can use the short name:

    ```bash
    kubectl get ep
    ```

    The output should be similar to this one:

    ```bash
    NAME                 ENDPOINTS                                                        AGE
    kubernetes           188.185.125.181:6443                                             132d
    nginx-clusterip      10.100.126.79:80,10.100.126.80:80,10.100.126.81:80 + 7 more...   64d
    ```

    As we've already mentioned, Kubernetes will automatically create an Endpoints and EndpointSlice object with the same name as the Service.

- To view more details about an **Endpoint** object:

    - Use `kubectl get` command:

        ```bash
        kubectl get endpoints nginx-clusterip -o yaml
        ```

        The output should be similar to this one:

        ```yaml
        apiVersion: v1
        kind: Endpoints
        metadata:
            name: nginx-clusterip
            namespace: default
            selfLink: /api/v1/namespaces/default/endpoints/nginx-clusterip
            uid: b2fc6564-9281-46d8-ae66-6cdf50d63a05
        subsets:
        - addresses:
          - ip: 10.100.126.79
            nodeName: cms-daq-workshop-gml7jxg5oxyf-node-1
            targetRef:
            kind: Pod
            name: nginx-deployment-66b6c48dd5-wn9v6
            namespace: default
            resourceVersion: "37641063"
            uid: 47a57d1f-91a8-46d9-8e4c-48e066a13bf3
          - ip: 10.100.126.80
            nodeName: cms-daq-workshop-gml7jxg5oxyf-node-1
            targetRef:
            kind: Pod
            name: nginx-deployment-66b6c48dd5-4ckx7
            namespace: default
            resourceVersion: "37641120"
            uid: 1d082584-d4ca-48fe-90aa-43e0937fde35

        ...
        ```

    - _Or_ use the `kubectl describe` command:

        ```bash
        kubectl describe endpoints nginx-clusterip
        ```

        The output should be similar to this:

        ```bash
        Name:         nginx-clusterip
        Namespace:    default
        Labels:       <none>
        Subsets:
        Addresses:          10.100.126.79,10.100.126.80,10.100.126.81,10.100.126.82,10.100.126.83,10.100.155.149,10.100.155.150,10.100.155.151,10.100.155.153,10.100.155.154
        NotReadyAddresses:  <none>
        Ports:
            Name  Port  Protocol
            ----  ----  --------
            http  80    TCP

        Events:  <none>
        ```
    
    The addresses listed should match the IPv4 addresses assigned to the Pods matching the label selector of the Service.

- To view details about the corresponding **EndpointSlice** object, we can use the `kubectl describe` and the `kubectl get` commands in a similar manner:

    ```bash
    kubectl describe endpointslice nginx-clusterip-pxbh4
    ```

    The output should look like this:

    ```bash
    Name:         nginx-clusterip-pxbh4
    Namespace:    default
    Labels:       endpointslice.kubernetes.io/managed-by=endpointslice-controller.k8s.io
                kubernetes.io/service-name=nginx-clusterip
    Annotations:  endpoints.kubernetes.io/last-change-trigger-time: 2023-04-20T09:14:43Z
    AddressType:  IPv4
    Ports:
    Name  Port  Protocol
    ----  ----  --------
    http  80    TCP
    Endpoints:
    - Addresses:  10.100.126.79
        Conditions:
        Ready:    true
        Hostname:   <unset>
        TargetRef:  Pod/nginx-deployment-66b6c48dd5-wn9v6
        NodeName:   cms-daq-workshop-gml7jxg5oxyf-node-1
        Zone:       cern-geneva-b
    - Addresses:  10.100.155.149
        Conditions:
        Ready:    true
        Hostname:   <unset>
        TargetRef:  Pod/nginx-deployment-66b6c48dd5-89fsc
        NodeName:   cms-daq-workshop-gml7jxg5oxyf-node-0
        Zone:       cern-geneva-b
    
    ...
    ```

!!! note
    Kubernetes will automatically update the Endpoint resource IPs in case of editing or scaling of a Deployment in which Pods are already linked to a Service.


## Example - Termination Behavior for Pods And Their Endpoints

As we already mentioned, the Endpoint object is refreshed with a new list of endpoints when:

- A Pod is created.
- A Pod is deleted.
- A label is modified on the Pod.

This example is to illustrate the flow of Pod termination in connection with the corresponding endpoint state and removal by using a simple nginx web server to demonstrate the concept:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-grace-endpoints
  labels:
    app: nginx-grace-endpoints
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-grace-endpoints
  template:
    metadata:
      labels:
        app: nginx-grace-endpoints
    spec:
      terminationGracePeriodSeconds: 60 # extra long grace period
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        lifecycle:
          preStop:
            exec:
              # Real life termination may take any time up to terminationGracePeriodSeconds.
              # In this example - just hang around for at least the duration of terminationGracePeriodSeconds,
              # at 60 seconds container will be forcibly terminated.
              # Note, all this time nginx will keep processing requests.
              command: [
                "/bin/sh", "-c", "sleep 180"
              ]

---

apiVersion: v1
kind: Service
metadata:
  name: nginx-grace-endpoints
spec:
  selector:
    app: nginx-grace-endpoints
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

Confirm that there is an EndpointSlice object for the created Service:

```bash
kubectl get endpointslices -o yaml -l kubernetes.io/service-name=nginx-grace-endpoints
```

Then terminate the nginx Pod that is part of the created Deployment and that is respecting the graceful termination period configuration:

```bash
kubectl delete pod <POD_NAME>
```

At the same time (maybe in another terminal window/session):

- view the pods - using `kubectl get pods -l app=nginx-grace-endpoints`
- view how the EndpointSlice object behaves - using `kubectl get endpointslices -o yaml -l kubernetes.io/service-name=nginx-grace-endpoints`

While one of the Pods is in a Terminating state, the corresponding EndpointSlice object should look like this:

```diff
  apiVersion: v1
  kind: EndpointSlice
  metadata:
    name: nginx-grace-endpoints-9nvk8
    namespace: default
  items:
  - addressType: IPv4
    apiVersion: discovery.k8s.io/v1
    endpoints:
    - addresses:
-     - 10.100.245.134
-     conditions:
-       ready: false
-       serving: true
-       terminating: true
      nodeName: cms-daq-workshop-gml7jxg5oxyf-node-2
      targetRef:
        kind: Pod
        name: nginx-grace-endpoints-57444f4c6f-t6kff
        namespace: default
        resourceVersion: "39016019"
        uid: e86b056d-930b-4740-8f75-07be558d0b9e
      zone: cern-geneva-b
    - addresses:
+     - 10.100.245.135
+     conditions:
+       ready: true
+       serving: true
+       terminating: false
      nodeName: cms-daq-workshop-gml7jxg5oxyf-node-2
      targetRef:
        kind: Pod
        name: nginx-grace-endpoints-57444f4c6f-976r7
        namespace: default
        resourceVersion: "39016045"
        uid: f3a49202-6653-4be9-9b57-879799300360
      zone: cern-geneva-b
```

!!! note
    Note that the Endpoints object already has removed the IP address of the terminating Pod from its list of addresses, while the EndpointSlices object keeps the IP of the terminating Pod but contains more Pod conditions, e.g. if the Pod is ready, terminating and/or serving traffic.

    You can confirm that by using the `kubectl get endpoints <NAME> -o yaml` and the `kubectl get endpointslices -l kubernetes.io/service-name=nginx-grace-endpoints -o yaml` commands.
