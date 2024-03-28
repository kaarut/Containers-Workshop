# Configuration Best Practices

- Don't specify default values unnecessarily: simple, minimal configuration will make errors less likely.
- **Create a Service before its corresponding backend workloads** (Deployments or ReplicaSets), and before any workloads that need to access it. When Kubernetes starts a container, it provides environment variables pointing to all the Services which were running when the container was started. For example, if a Service named `foo` exists, all containers will get the following variables in their initial environment:

    ```bash
    FOO_SERVICE_HOST=<the host the Service is running on>
    FOO_SERVICE_PORT=<the port the Service is running on>
    ```

    _This does imply an ordering requirement_ - **any `Service` that a `Pod` wants to access must be created before the `Pod` itself, or else the environment variables will not be populated**. DNS does not have this restriction.

- An optional (though strongly recommended) cluster add-on is a DNS server. The DNS server watches the Kubernetes API for new `Services` and creates a set of DNS records for each. If DNS has been enabled throughout the cluster then all `Pods` should be able to do name resolution of `Services` automatically.

- Don't specify a `hostPort` for a Pod unless it is absolutely necessary. When you bind a Pod to a `hostPort`, it limits the number of places the Pod can be scheduled, because each <`hostIP`, `hostPort`, `protocol`> combination must be unique. If you don't specify the `hostIP` and `protocol` explicitly, Kubernetes will use `0.0.0.0` as the default `hostIP` and `TCP` as the default `protocol`.

    If you only need access to the port for debugging purposes, you can use the [apiserver proxy](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/#manually-constructing-apiserver-proxy-urls) or `kubectl port-forward`.

    If you explicitly need to expose a Pod's port on the node, consider using a `NodePort` Service before resorting to `hostPort`.

- Avoid using hostNetwork, for the same reasons as `hostPort`.

- Use [headless Services](../000-services/headless.md) (which have a `ClusterIP` of `None`) for service discovery when you don't need `kube-proxy` load balancing.
