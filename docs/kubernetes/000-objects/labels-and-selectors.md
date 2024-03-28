# Labels and Selectors

Labels are key/value pairs that are attached to objects, such as pods. Labels are intended to be used to specify identifying attributes of objects that are meaningful and relevant to users, but do not directly imply semantics to the core system. Labels can be used to organize and to select subsets of objects.

Labels allow for efficient queries and watches and are ideal for use in UIs and CLIs. **Non-identifying information should be recorded using [annotations](./annotations.md)**.


For the label keys, the `kubernetes.io/` and `k8s.io/` prefixes are reserved for Kubernetes core components.


Example labels:

- `"release" : "stable"`, `"release" : "canary"`
- `"environment" : "dev"`, `"environment" : "qa"`, `"environment" : "production"`
- `"tier" : "frontend"`, `"tier" : "backend"`, `"tier" : "cache"`


For example, here's the configuration file for a Pod that has two labels `environment: production` and `app: nginx`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: label-demo
  labels:
    environment: production
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

## Label selectors

Unlike names and UIDs, labels do not provide uniqueness. In general, we expect many objects to carry the same label(s).

A label selector can be made of multiple _requirements_ which are comma-separated. In the case of multiple requirements, all must be satisfied so the comma separator acts as a logical _AND_ (`&&`) operator.

The API currently supports two types of selectors: _equality-based_ and _set-based_.

### Equality-based selectors

To list Pod resources that are labeled `environment=production` _and_ `tier=frontend`:

```bash
kubectl get pods -l environment=production,tier=frontend
```

### Set-based selectors

To list Pod resources that are labeled `environment=production` _and_ `tier=frontend`:

```bash
kubectl get pods -l 'environment in (production),tier in (frontend)'
```

One advantage of the set-based selectors is that set-based requirements are more expressive. For instance, they can implement the _OR_ operator on values:

```bash
kubectl get pods -l 'environment in (production, qa)'
```


### Show labels with kubectl

Many Kubernetes objects support labels. You can view these labels with the `--show-labels` flag. For example, to show labels for all Pods:

```bash
kubectl get pods -n kube-system --show-labels
```
