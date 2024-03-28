# Namespaces

In Kubernetes, **_namespaces_ provides a mechanism for isolating groups of resources within a single cluster**. Names of resources need to be unique within a namespace, but not across namespaces. Namespace-based scoping is applicable only for namespaced objects (e.g. Deployments, Services, etc) and not for cluster-wide objects (e.g. StorageClass, Nodes, PersistentVolumes, etc).

Namespaces cannot be nested inside one another.

!!! note
    Kubernetes' namespaces shouldn't be confused with [Linux namespaces](https://en.wikipedia.org/wiki/Linux_namespaces), which are used to partition kernel resources for containers isolation.


## When to Use Multiple Namespaces

Namespaces are intended for use in environments with many users spread across multiple teams, or projects. For clusters with a few to tens of users, you should not need to create or think about namespaces at all.

The simple clean-up process is central to managing operations at scale and the cluster management add-ons such as role-based access control (RBAC), making namespaces even more powerful.

Namespaces are a way to divide cluster resources between multiple users (via resource quota).

!!! note
    For a production cluster, consider not using the `default` namespace.


## Initial namespaces

Kubernetes starts with four initial namespaces:

- `default`
- `kube-public`
- `kube-node-lease`
- `kube-system`

!!! tip
    Avoid creating namespaces with the prefix `kube-`, since it is reserved for Kubernetes system namespaces.


## Viewing namespaces

You can list the current namespaces in a cluster using:

```bash
kubectl get namespace
```

_or_ use the shortname:

```bash
kubectl get ns
```


## Setting the namespace for a request

To set the namespace for a current request, use the --namespace flag:

```bash
kubectl get pods --namespace=<NAMESPACE_NAME>
```

## Not all objects are in a namespace

Most Kubernetes resources (e.g. pods, services, replication controllers, and others) are in some namespaces.

To see which Kubernetes resources are and aren't in a namespace:

```bash
# In a namespace
kubectl api-resources --namespaced=true

# Not in a namespace
kubectl api-resources --namespaced=false
```

## Namespaces and DNS

When you create a Service, it creates a corresponding DNS entry. This entry is of the form `<service-name>.<namespace-name>.svc.cluster.local`, which means that if a container only uses `<service-name>`, it will resolve to the service which is local to a namespace. 

As a result, all namespace names must be valid RFC 1123 DNS labels, which means that the namespace name must:

- contain at most 63 characters
- contain only lowercase alphanumeric characters or '-'
- start with an alphanumeric character
- end with an alphanumeric character
