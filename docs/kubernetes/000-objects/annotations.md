# Annotations

You can use Kubernetes annotations to **attach arbitrary non-identifying metadata to objects**. **Clients such as tools and libraries can retrieve this metadata**.

Annotations, like labels, are key/value maps.


# Labels vs Annotations

You can use either labels or annotations to attach metadata to Kubernetes objects. Labels can be used to select objects and to find collections of objects that satisfy certain conditions. In contrast, annotations are not used to identify and select objects. The metadata in an annotation can be small or large, structured or unstructured, and can include characters not permitted by labels.

## Use-cases

Here are some examples of information that could be recorded in annotations:

- Build, release, or image information like timestamps, release IDs, git branch, PR numbers, image hashes, and registry address.
- Pointers to logging, monitoring, analytics, or audit repositories.
- Client library or tool information that can be used for debugging purposes: for example, name, version, and build information.
- User or tool/system provenance information, such as URLs of related objects from other ecosystem components.
- Lightweight rollout tool metadata: for example, config or checkpoints.
- Phone or pager numbers of persons responsible, or directory entries that specify where that information can be found, such as a team web site.
- Directives from the end-user to the implementations to modify behavior or engage non-standard features.


## Syntax and character set

_Annotations_ are key/value pairs. Valid annotation keys have two segments: an optional prefix and name, separated by a slash (`/`).

!!! note
    The `kubernetes.io/` and `k8s.io/` prefixes are reserved for Kubernetes core components.

An example of a Pod using annotations:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: annotations-demo
  annotations:
    app.kubernetes.io/owner: "thrizopo"
    app.kubernetes.io/repository: "https://gitlab.cern.ch/cms-daq-sysadmins/containers-workshop"
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

## Kubectl

### Querying by annotations

Although kubectl has the native ability to query on labels, it does not provide the same support for annotations.

You can, however, use the power of jsonpath to find these objects. Below is an example of finding all Services that have the `prometheus.io/scrape` annotation:

```bash
# service that has annotation, regardless of value
kubectl get service -A -o jsonpath='{.items[?(@.metadata.annotations.prometheus\.io/scrape)].metadata.name}'

# has annotation set to "true"
kubectl get service -A -o jsonpath='{.items[?(@.metadata.annotations.prometheus\.io/scrape=="true")].metadata.name}'
```

### Annotate

To add an annotation to a Pod:

```bash
kubectl annotate pods my-pod icon-url=http://goo.gl/XXBTWq
```