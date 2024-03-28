# Pod's DNS Policy

## dnsPolicy

DNS policies can be set on a per-Pod basis. Currently Kubernetes supports the following Pod-specific DNS policies. These policies are specified in the `dnsPolicy` field of a Pod Spec.

- `Default`: The Pod inherits the name resolution configuration from the node that the Pods run on.
- `ClusterFirst`: Any DNS query that does not match the configured cluster domain suffix, such as `www.kubernetes.io`, is forwarded to an upstream nameserver by the DNS server. Cluster administrators may have extra stub-domain and upstream DNS servers configured.
- `ClusterFirstWithHostNet`: For Pods running with hostNetwork, you should explicitly set its DNS policy to `ClusterFirstWithHostNet`. Otherwise, Pods running with hostNetwork and `ClusterFirst` will fallback to the behavior of the `Default` policy.
- `None`: It allows a Pod to ignore DNS settings from the Kubernetes environment. All DNS settings are supposed to be provided using the `dnsConfig` field in the Pod Spec.

!!! note
    `Default` is **not** the default DNS policy. If `dnsPolicy` is not explicitly specified, then `ClusterFirst` is used.

## Example

The example below shows a Pod with its DNS policy set to "`ClusterFirstWithHostNet`" because it has `hostNetwork` set to `true`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
spec:
  containers:
  - image: busybox:1.28
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
    name: busybox
  restartPolicy: Always
  hostNetwork: true
  dnsPolicy: ClusterFirstWithHostNet
```
