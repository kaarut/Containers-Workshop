# Kubernetes Objects

**Kubernetes objects are persistent entities in the Kubernetes system**.

An object is a _"record of intent"_ – once created, the cluster does its best to ensure it exists as defined. This is known as the cluster’s _"desired state"_.

Kubernetes is always working to make an object’s "current state" equal to the object’s "desired state". A desired state can describe:

- What pods (containers) are running, and on which nodes
- The resources available to those applications
- How many replicas of a container are running
- The policies around how those applications behave, such as restart policies, upgrades, and fault-tolerance

## Object spec and status

Almost every Kubernetes object includes two nested object fields that govern the object's configuration: the object _`spec`_ and the object _`status`_:

  - For objects that have a `spec`, you have to set this when you create the object, providing a description of the characteristics you want the resource to have: its _desired state_.
  - The `status` describes the _current state_ of the object, supplied and updated by the Kubernetes system and its components. The Kubernetes control plane continually and actively manages every object's actual state to match the desired state you supplied.


## Example Kubernetes Object

**Most often, you provide the information to `kubectl` in a `.yaml` file**.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

One way to create objects is using a `.yaml` file is to use the [`kubectl apply`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply) command, passing the `.yaml` file as an argument. For example:

```bash
kubectl apply -f deployment.yaml
```

### Required fields

In the `.yaml` file for the Kubernetes object you want to create, you'll need to set values for the following fields:

- `apiVersion` - Which version of the Kubernetes API you're using to create this object.
- `kind` - What kind of object you want to create.
- `metadata` - Data that helps uniquely identify the object, including a `name` string, `UID`, and optional `namespace`.
- `spec` - What state you desire for the object.


The precise format of the object spec is different for every Kubernetes object, and contains nested fields specific to that object. The [Kubernetes API Reference](https://kubernetes.io/docs/reference/kubernetes-api/) can help you find the spec format for all of the objects you can create using Kubernetes.

## List API Resources

To list the API resources that are available:

```bash
kubectl api-resources
```


