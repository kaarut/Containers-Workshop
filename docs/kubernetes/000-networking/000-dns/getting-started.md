# DNS in Kubernetes

**Kubernetes creates DNS records for Services and Pods**. You can contact Services with consistent DNS names instead of IP addresses.

What objects get DNS records?

1. Services
2. Pods

By default, a client Pod's DNS search list includes the Pod's own namespace and the cluster's default domain.


## Service Discovery

**CoreDNS implements the Kubernetes DNS spec**.

CoreDNS is deployed in a Kubernetes cluster as a Deployment and exposed as a ClusterIP service. This means that all communications with the DNS service inside a cluster are subject to the same network forwarding rules used and limitations experienced by normal Pods and set up by the CNI and Services plugins.

!!! info
    The entire CoreDNS configuration can be found in the `coredns` ConfigMap (usually under the `kube-system` namespace).


## Namespaces of Services

A DNS query may return different results based on the namespace of the Pod making it. DNS queries that don't specify a namespace are limited to the Pod's namespace. Access Services in other namespaces by specifying it in the DNS query.

- For example, consider a Pod in a `test` namespace.
- A `data` Service is in the `prod` namespace.
- A query for `data` returns no results, because it uses the Pod's `test` namespace.
- A query for `data.prod` returns the intended result, because it specifies the namespace.

DNS queries may be expanded using the Pod's `/etc/resolv.conf`. Kubelet configures this file for each Pod. For example, a query for just `data` may be expanded to `data.test.svc.cluster.local`. The values of the search option are used to expand queries.

Below is an example of an `/etc/resolv.conf` file:

```bash
nameserver 10.254.0.10
search <namespace>.svc.cluster.local svc.cluster.local cluster.local cern.ch
options ndots:5
```

In summary, a Pod in the `test` namespace can successfully resolve either `data.prod` or `data.prod.svc.cluster.local`.

The search domains and `ndots` value are configured so that any non-FQDN DNS query made by a Pod is first tried in all of the specified domains, which allows for internal cluster DNS schema to take precedence over the external DNS.

The downside of this behaviour is that any external domain lookup will require at least 4 separate queries.

## DNS Records

## Services

As it is mentioned in the Services chapter, DNS is an essential part of how Services are consumed by end clients.

#### A/AAAA records


**All Kubernetes Services have at least one corresponding A/AAAA DNS record** in the format of `{service-name}.{namespace}.svc.{cluster-domain}` and the response format depends on the type of a Service:

| Service Type | Response |
|:------------:|:--------:|
| `ClusterIP`, `NodePort`, `LoadBalancer` | ClusterIP value |
| `Headless` | List of Endpoint IPs selected by the Service |
| `ExternalName` | CNAME pointing to the value of `spec.externalName` |

### Pods

#### A/AAAA records

In general a Pod has the following DNS resolution: `pod-ip-address.my-namespace.pod.cluster-domain.example`.

For example, if a Pod in the `default` namespace has the IP address `172.17.0.3`, and the domain name for your cluster is `cluster.local`, then the Pod has a DNS name: `172-17-0-3.default.pod.cluster.local`.

Any Pods exposed by a Service have the following DNS resolution available: `pod-ip-address.service-name.my-namespace.svc.cluster-domain.example`.


## External DNS

The **[DNS Specification](https://github.com/kubernetes/dns/blob/master/docs/specification.md) is only focused on the intra-cluster DNS resolution and service discovery**. Anything to do with external DNS is left out of scope. For end-users that are located outside of the cluster, Kubernetes has to provide a way to discover external Kubernetes resources, LoadBalancer Services, Ingresses and Gateways, and there are two ways this can be accomplished:

- An out-of-cluster DNS zone can be orchestrated by the [**ExternalDNS** cluster add-on](https://github.com/kubernetes-sigs/external-dns) – a Kubernetes controller that synchronises external Kubernetes resources with any supported third-party DNS provider via an API (see the [GitHub page](https://github.com/kubernetes-sigs/external-dns#externaldns) for the list of supported providers).
- An existing DNS zone can be configured to delegate a subdomain to a self-hosted external DNS plugin. This approach assumes that this DNS plugin is deployed inside a cluster and exposed via a LoadBalancer IP, which is then used in an NS record for the delegated zone. All queries hitting this subdomain will get forwarded to this plugin which will respond as an authoritative nameserver for the delegated subdomain.
