# DNS in Kubernetes - Extras

## DNS Resolver Configuration

Below is an example of an `/etc/resolv.conf` file:

```bash
nameserver 10.254.0.10
search <namespace>.svc.cluster.local svc.cluster.local cluster.local cern.ch
options ndots:5
```

As you can see there are three directives:

- The nameserver IP is the Kubernetes service IP of `coredns`
- There are 4 local `search` domains specified
- There’s a `ndots:5` option

In order to understand how the local `search` domains and `ndots:5` settings play together, we need to understand how the DNS resolution works for non fully qualified names.


### What is a fully qualified name?

A fully qualified name is a name for which no local search will be executed and the name will be treated as an absolute one during the resolution. By convention, DNS software consider a name fully qualified if ends with a full stop (.) and non fully qualified otherwise. Ie. `google.com.` is fully qualified, `google.com` is not.


### How non fully qualified name resolution is performed?

When an application connects to a remote host specified by name, a DNS resolution is performed typically via a syscall, like getaddrinfo(). If the name is not fully qualified (not ending with a .), will the syscall try to resolve the name as an absolute one first, or will go through the local search domains first? It depends on the ndots option.

From the `resolv.conf` man-page:

> `ndots:n`
> Sets a threshold for the number of dots which must 
> appear in a name given to res_query before an initial
> absolute query will be made.
> The default for n is 1, meaning that if
> there are any dots in a name, the name will be
> tried first as an absolute name before any search
> list elements are appended to it.  The value for
> this option is silently capped to 15.

This means that if `ndots` is set to `5` and the name contains less than 5 dots inside it, the syscall will try to resolve it sequentially going through all local search domains first and - in case none succeed - will resolve it as an absolute name only at last.


### Why `ndots:5` can negatively affect application performances?

As you can understand, if your application does a lot of external traffic, for each TCP connection established (or more specifically, for each name resolved) it will issue 5 DNS queries before the name is correctly resolved, because it will go through the 4 local search domains first and will finally issue an absolute name resolution query.

For example, any Pod in the `default` Namespace, can lookup the ClusterIP of the `kubernetes` Service in a single lookup

```bash
kubectl exec -it -n default <POD_NAME> -- nslookup kubernetes
```

Running `stern` will help us get logs for all the CoreDNS pods (and then we can filter results based on the desired `grep` pattern):

!!! info
    [stern](https://github.com/stern/stern) is a multi pod and container log tailing for Kubernetes.


```bash
$ stern -l k8s-app=kube-dns | grep "kubernetes"

coredns-5b6d5d5566-rch9f coredns [INFO] 10.100.155.171:39718 - 12740 "A IN kubernetes.default.svc.cluster.local. udp 54 false 512" NOERROR qr,aa,rd 106 0.000252005s
coredns-5b6d5d5566-rch9f coredns [INFO] 10.100.155.171:38589 - 2358 "AAAA IN kubernetes.default.svc.cluster.local. udp 54 false 512" NOERROR qr,aa,rd 147 0.000127724s
```

Let's see what happens if we try to perform an external domain lookup. In this example we'll try to resolve [`home.cern`](https://home.cern/):

```bash
$ kubectl exec -it <POD_NAME> -- nslookup home.cern
```

The logs from the CoreDNS pods:

```bash
$ stern -l k8s-app=kube-dns | grep "home.cern"

coredns-5b6d5d5566-g8rr7 coredns [INFO] 10.100.155.171:35931 - 4660 "A IN home.cern.default.svc.cluster.local. udp 53 false 512" NXDOMAIN qr,aa,rd 146 0.000142282s
coredns-5b6d5d5566-rch9f coredns [INFO] 10.100.155.171:58356 - 30101 "A IN home.cern.svc.cluster.local. udp 45 false 512" NXDOMAIN qr,aa,rd 138 0.000168419s
coredns-5b6d5d5566-g8rr7 coredns [INFO] 10.100.155.171:39167 - 44098 "A IN home.cern.cluster.local. udp 41 false 512" NXDOMAIN qr,aa,rd 134 0.000110054s
coredns-5b6d5d5566-g8rr7 coredns [INFO] 10.100.155.171:40425 - 19518 "A IN home.cern.cern.ch. udp 35 false 512" NXDOMAIN qr,aa,rd,ra 114 0.000076875s
coredns-5b6d5d5566-g8rr7 coredns [INFO] 10.100.155.171:46422 - 13378 "A IN home.cern. udp 27 false 512" NOERROR qr,rd,ra 132 0.000651324s
coredns-5b6d5d5566-g8rr7 coredns [INFO] 10.100.155.171:43849 - 45561 "AAAA IN home.cern. udp 27 false 512" NOERROR qr,rd,ra 144 0.000418275s
```

As you can see it will go through the 4 local search domains first and will finally issue an absolute name resolution query.


### Solution #1 - use fully qualified names

If you have few static external names (ie. defined in the application config) to which you create a large number of connections, the easiest solution is probably switching them to fully qualified, just adding a `.` at the end.


### Solution #2 - customize ndots with dnsConfig

Kubernetes has a feature that allows more control on the DNS settings for a Pod via the `dnsConfig` pod property. Among the other things, it allows to customize the `ndots` value for a specific pod, ie.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dns-example
spec:
  containers:
    - name: test
      image: nginx
  dnsConfig:
    options:
      - name: ndots
        value: "1"
```

## CoreDNS - Server-side

CoreDNS implements the Kubernetes DNS spec in a [dedicated plugin](https://coredns.io/plugins/kubernetes/) that gets compiled into a static binary and deployed in a Kubernetes cluster as a Deployment and exposed as a ClusterIP service. This means that all communications with the DNS service inside a cluster are subject to the same network forwarding rules used and limitations experienced by normal Pods and set up by the CNI and Services plugins.

Since DNS speed and stability are considered crucial in any network-based communication, CoreDNS implementation is [highly optimised](https://github.com/coredns/deployment/blob/master/kubernetes/Scaling_CoreDNS.md) to minimise memory consumption and maximise query processing rate. In order to achieve that, CoreDNS stores only the relevant parts of Services, Pods and Endpoints objects in its local cache that is optimised to return a response in a single lookup.

By default, CoreDNS also acts as a DNS proxy for all external domains (e.g. example.com) using the [`forward` plugin](https://coredns.io/plugins/forward/) and is often deployed with the [`cache` plugin](https://coredns.io/plugins/cache/) enabled. The entire CoreDNS configuration can be found in the `coredns` ConfigMap (usually under the `kube-system` namespace):

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        log
        health
        kubernetes cluster.local 10.254.0.0/16 10.100.0.0/16 {
           pods verified
           fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        forward . /etc/resolv.conf
        cache 900
        loop
        reload
        loadbalance
    }
```

The `Corefile` is used o configure CoreDNS.

You can find more details on the used plugins in the [official CoreDNS documentaiton](https://coredns.io/plugins/).
