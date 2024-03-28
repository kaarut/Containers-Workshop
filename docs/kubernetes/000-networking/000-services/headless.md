# Headless Service

This type of service **does not perform any load-balancing and only implements DNS Service Discovery**, based on the Kubernetes [DNS Spec](https://github.com/kubernetes/dns/blob/master/docs/specification.md#24---records-for-a-headless-service).

Although this is the simplest and the most basic type of Service, its use is **mainly limited to stateful applications like databases and clusters**. In these use case the assumption is that clients have some prior knowledge about the application they’re going to be communicating with, e.g. number of nodes, naming structure, and can handle failover and load-balancing on their own.

Some typical examples of stateful applications that use this kind of service are:

- [zookeeper](https://github.com/bitnami/charts/blob/main/bitnami/zookeeper/templates/svc-headless.yaml)
- [etcd](https://github.com/bitnami/charts/blob/main/bitnami/etcd/templates/svc-headless.yaml)
- [consul](https://github.com/hashicorp/consul-helm/blob/master/templates/server-service.yaml)


The only thing that makes a service "Headless" is the `clusterIP: None` which, on the one hand, tells dataplane agents to ignore this resource and, on the other hand, tells the DNS plugin that it needs special type of processing. The rest of the API parameters look similar to any other Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
spec:
  clusterIP: None
  selector:
    app: nginx
  ports:
  - name: http
    protocol: TCP
    port: 80
```


## DNS Resolution

Let's create a temporary Ubuntu Pod, run `nslookup` and see where the Service resolves to.

```bash
# Create a temporary Ubuntu Pod
$ kubectl run temp-ubuntu -it --rm --image=ubuntu -- bash

# Run nslookup **inside the Ubuntu Pod** to see
# what the Headless Service resolves to.
$ nslookup nginx-headless.default.svc.cluster.local
```

!!! note
    The first part is (i.e. `nginx-headless.default.svc.cluster.local`) is:

    - `nginx-headless`: the Service name (as stated on the `.metadata.name` field).
    - `default`: the Namespace that the Service belongs to.

    Ignore the `svc.cluster.local` suffix for now. We're going to cover it in the DNS chapter.

The last command should return something like this:

```bash
Server:		10.254.0.10
Address:	10.254.0.10#53

Name:	nginx-headless.default.svc.cluster.local
Address: 10.100.245.154
Name:	nginx-headless.default.svc.cluster.local
Address: 10.100.126.93
Name:	nginx-headless.default.svc.cluster.local
Address: 10.100.155.174
Name:	nginx-headless.default.svc.cluster.local
Address: 10.100.126.95
Name:	nginx-headless.default.svc.cluster.local
Address: 10.100.245.155
Name:	nginx-headless.default.svc.cluster.local
Address: 10.100.155.173
Name:	nginx-headless.default.svc.cluster.local
Address: 10.100.155.171
Name:	nginx-headless.default.svc.cluster.local
Address: 10.100.155.172
Name:	nginx-headless.default.svc.cluster.local
Address: 10.100.126.94
Name:	nginx-headless.default.svc.cluster.local
Address: 10.100.245.153
```

As you can see, the `nslookup` command returned multiple DNS A records. In our case, the `nginx-headless.default.svc.cluster.local` name resolves to ten IP addresses (`10.100.245.154`, `10.100.126.93`, etc.), which correspond to the IP addresses of the nginx Pods.

Therefore, when there is no need of load-balancing or single-service IP addresses, we create a `Headless` Service which is used for creating a service grouping. That does not allocate an IP address or forward traffic. Clients can therefore do a simple DNS A record lookup and get the IPs of all the pods that are part of the Service. The client can then use that information to connect to one, many, or all of them.
