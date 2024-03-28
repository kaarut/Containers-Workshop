# hostNetwork

There are several ways how to expose your application running on the Kubernetes cluster to the outside world, as we already discussed in the [Services](../000-services/getting-started.md) and [Ingress](../000-ingress/getting-started.md) sections. Another option is specifying the Pods running in the host network.

When a Pod is configured with `hostNetwork: true`, the applications running in such a Pod can directly see and the network interfaces of the host machine where the Pod was started, which means that the Pod can use the network namespace and network resources of the node. An application that is configured to listen on all network interfaces will in turn be accessible on all network interfaces of the host machine. Here is an example definition of a Pod that uses host networking:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: influxdb-hostnetwork
spec:
  hostNetwork: true
  containers:
  - name: influxdb
    image: influxdb
```

Once the above manifest is applied to your Kubernetes cluster, a Pod should be scheduled in one of the worker nodes:

```bash
kubectl describe pods influxdb-hostnetwork
```

The output should be similar to this:

```bash
Name:         influxdb-hostnetwork
Namespace:    default
Priority:     0
Node:         cms-daq-workshop-gml7jxg5oxyf-node-2/188.185.125.110
```

As you can see, the Pod has been placed on the `cms-daq-workshop-gml7jxg5oxyf-node-2` worker node with IPv4 `188.185.125.110`. Once the Influxdb container is up-and-running, we should be able to perform HTTP HEAD requests to it on the `/ping` endpoint and InfluxDB should respond with HTTP 204 No Content when working properly (make sure to replace `188.185.125.110` with your worker's node IP address):

```bash
$ curl -I http://188.185.125.110:8086/ping

HTTP/1.1 204 No Content
Vary: Accept-Encoding
X-Influxdb-Build: OSS
X-Influxdb-Version: v2.7.0
Date: Fri, 21 Apr 2023 15:11:48 GMT
```

!!! note
    - The `8086` port if the default port for InfluxDB.

    - The `/ping` endpoint accepts both `GET` and `HEAD` requests and is used to check the status of the InfluxDB instance. More details on the [official InfluxDB documentation](https://docs.influxdata.com/influxdb/v1.3/tools/api/#ping).

If you try to make an HTTP HEAD request to any other worker node of your cluster (except from the one that runs the InfluxDB Pod), we'll get an error. 

```bash
$ curl -I http://188.185.124.201:8086/ping

curl: (7) Failed to connect to 188.185.124.201 port 8086 after 14 ms: Connection refused
```

In this case, we don't get any of the proxying benefits of kube-proxy.

Note that every time the Pod is restarted Kubernetes can reschedule the Pod onto a different node and so the application will change its IP address. Besides that two applications requiring the same port cannot run on the same node. This can lead to port conflicts when the number of applications running on the cluster grows.


## What is the host networking good for

For cases where a direct access to the host networking is required. For example, the Kubernetes networking plugin Flannel can be deployed as a daemon set on all nodes of the Kubernetes cluster. Due to `hostNetwork: true` the Flannel has full control of the networking on every node in the cluster allowing it to manage the overlay network to which the pods with `hostNetwork: false` are connected to.

!!! danger "Warning"
    Don't specify `hostNetwork: true` for a Pod unless it is absolutely necessary.
