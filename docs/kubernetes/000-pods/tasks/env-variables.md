# Environment Variables

Kubernetes allows to define environment variables for a container in a Pod.

## Define an environment variable for a container

When you create a Pod, you can set environment variables for the containers that run in the Pod. To set environment variables, include the `env` or `envFrom` field in the configuration file.

Let's have a look at a simple example.

- Apply the following Pod object in your Kubernetes cluster:

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx-env-variables
    spec:
      containers:
      - name: nginx
        image: nginx
        env:
        - name: CERN_PROJECT
          value: "cms-daq"
        - name: CERN_BUILDING
          value: "B40"
    ```

- Verify that the Pod is up and running:

    ```bash
    kubectl get pods nginx-env-variables
    ```

- Print environment variables of your container:

    ```bash
    kubectl exec -it nginx-env-variables -- env | grep -i cern
    ```

    The output of this command should return the environment variables we set to the container, when creating the Pod on the first step.

    !!! note
        If your Pod contains multiple containers, you can use the `-c` flag to provide the container name that you want the command to be executed at.


!!! danger
    **Do NOT set secrets (or any other sensitive information) for the containers in a Pod using the `env[].value` field!** Anyone that has access to the `kubectl get pods` command (through RBAC), will be able to view these secret/sensitive values in plaintext (e.g. when running `kubectl get pods POD_NAME -o yaml`).

    Instead, a [`Secret` object](https://kubernetes.io/docs/concepts/configuration/secret/) should be created and then use the `env[].valueFrom.secretKeyRef` field to reference the secret. We'll have a look at an example later in the workshop, once we introduce [`Secrets`](./../../000-configuration/secrets.md).


## Expose Pod Information to Containers Through Environment Variables

It is sometimes useful for a container to have information about itself, without being overly coupled to Kubernetes. The _downward API_ allows containers to consume information about themselves or the cluster without using the Kubernetes client or API server.

In this section we will have a look on how a Pod can use environment variables to expose information about itself to containers running in the Pod, using the _downward API_. You can use environment variables to expose Pod fields, container fields, or both.

In Kubernetes, there are two ways to expose Pod and container fields to a running container:

- Environment variables, as explained in this task
- Volume files

Together, these two ways of exposing Pod and container fields are called the downward API.


### Use Pod fields as values for environment variables

In this part of exercise, you create a Pod that has one container, and you project Pod-level fields into the running container as environment variables.


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dapi-envars-fieldref
spec:
  containers:
    - name: test-container
      image: registry.k8s.io/busybox
      command: [ "sh", "-c"]
      args:
      - while true; do
          echo -en '\n';
          printenv MY_NODE_NAME MY_POD_NAME MY_POD_NAMESPACE;
          printenv MY_POD_IP MY_POD_SERVICE_ACCOUNT;
          sleep 10;
        done;
      env:
        - name: MY_NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        - name: MY_POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: MY_POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: MY_POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: MY_POD_SERVICE_ACCOUNT
          valueFrom:
            fieldRef:
              fieldPath: spec.serviceAccountName
  restartPolicy: Never
```

In that manifest, you can see five environment variables. The `env` field is an array of environment variable definitions. The first element in the array specifies that the `MY_NODE_NAME` environment variable gets its value from the Pod's `spec.nodeName` field. Similarly, the other environment variables get their names from Pod fields.


Apply the above manifest and then verify that the container in the Pod is running:

```bash
# If the new Pod isn't yet healthy, rerun this command a few times.
kubectl get pods
```


View the container's logs:

```bash
kubectl logs dapi-envars-fieldref
```

The output shows the values of selected environment variables:

```bash
cms-daq-workshop-gml7jxg5oxyf-node-1
dapi-envars-fieldref
default
10.100.126.111
default
```


To see why these values are in the log, look at the command and args fields in the configuration file. When the container starts, it writes the values of five environment variables to stdout. It repeats this every ten seconds.

Next, get a shell into the container that is running in your Pod:

```bash
kubectl exec -it dapi-envars-fieldref -- sh
```

In your shell, view the environment variables:

```bash
# Run this in a shell inside the container
printenv
```

The output shows that certain environment variables have been assigned the values of Pod fields:

```
MY_POD_SERVICE_ACCOUNT=default
...
MY_POD_NAMESPACE=default
MY_POD_IP=10.100.126.111
...
MY_NODE_NAME=cms-daq-workshop-gml7jxg5oxyf-node-1
...
MY_POD_NAME=dapi-envars-fieldref
```

!!! info
    The available Kubernetes API fields that are available through the downward API can be found [here](https://kubernetes.io/docs/concepts/workloads/pods/downward-api/).


## Notes

- The environment variables set using the `env` or `envFrom` field **override** any environment variables specified in the container image.
- Environment variables may reference each other, however **ordering is important**. Variables making use of others defined in the same context must come later in the list. Similarly, avoid circular references.
