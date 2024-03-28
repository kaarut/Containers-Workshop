# Session Affinity

As we saw in previous sections, the traffic bound for the Service's IP:Port is proxied to an appropriate backend without the clients knowing anything about Kubernetes or Services or Pods.

If you want to make sure that connections from a particular client are passed to the same Pod each time, you can select the session affinity based on the client's IP addresses by setting `.spec.sessionAffinity` to `ClientIP` for a Service (the default is `None`).


## Session stickiness timeout

You can also set the maximum session sticky time by setting `.spec.sessionAffinityConfig.clientIP.timeoutSeconds` appropriately for a Service. (the default value is 10800, which works out to be 3 hours).

## Example

```yaml
kind: Service
apiVersion: v1
metadata:
  name: nginx-clusterip-session-affinity
spec:
  selector:
    app: nginx
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 80
  sessionAffinity: ClientIP
```

!!! note
    In the example above, the `.spec.ports[0].protocol` and the `.spec.ports[0].targetPort` could be omitted, as the default values can be used.

To specify the session stickiness timeout, this is what needs to be added:

```yaml
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10
```

Note that the example above would work hitting `ClusterIP` type service directly or with `Loadbalancer` type Service, but won't with an Ingress behind `NodePort` type Service. This is because with an Ingress, the requests come from many, randomly chosen source IP addresses.
