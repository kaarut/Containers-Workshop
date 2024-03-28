# imagePullPolicy

The `imagePullPolicy` for a container and the tag of the image affect when the kubelet attempts to pull (download) the specified image.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-image-pull-policy
  labels:
    app.kubernetes.io/name: nginx-image-pull-policy
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: nginx-image-pull-policy
  template:
    metadata:
      labels:
        app.kubernetes.io/name: nginx-image-pull-policy
    spec:
      containers:
      - name: nginx
        image: nginx:1.23
        imagePullPolicy: IfNotPresent
```

!!! tip
    Use the `kubectl describe pod` or the `kubectl get events` command to view events regarding image pulling.

Here's a list of the values you can set for `imagePullPolicy` and the effects these values have:

| Value | Description |
|:-----:|:-----------:|
| `IfNotPresent` | The image is pulled only if it is not already present locally. |
| `Always` | Every time the kubelet launches a container, the kubelet queries the container image registry to resolve the name to an image digest. If the kubelet has a container image with that exact digest cached locally, the kubelet uses its cached image; otherwise, the kubelet pulls the image with the resolved digest, and uses that image to launch the container. |
| `Never` | The kubelet does not try fetching the image. If the image is somehow already present locally, the kubelet attempts to start the container; otherwise, startup fails. |

!!! note
    **You should avoid using the `:latest` tag** when deploying containers in production as it is harder to track which version of the image is running and more difficult to roll back properly.

    Instead, specify a meaningful tag such as `v1.42.0`.

To make sure the Pod always uses the same version of a container image, you can specify the image's digest; replace `<image-name>:<tag>` with `<image-name>@<digest>` (for example, `image@sha256:45b23dee08af5e43a7fea6c4cf9c25ccf269ee113168c19722f87876677c5cb2`).

An image digest uniquely identifies a specific version of the image, so Kubernetes runs the same code every time it starts a container with that image name and digest specified.


## Default image pull policy

When you (or a controller) submit a new Pod to the API server, your cluster sets the imagePullPolicy field when specific conditions are met:

- if you omit the `imagePullPolicy` field, and the tag for the container image is `:latest`, `imagePullPolicy` is automatically set to `Always`.
- if you omit the `imagePullPolicy` field, and you don't specify the tag for the container image, `imagePullPolicy` is automatically set to `Always`.
- if you omit the `imagePullPolicy` field, and you specify the tag for the container image that isn't `:latest`, the `imagePullPolicy` is automatically set to `IfNotPresent`.


## Private Registries

Private registries may require keys to read images from them.

There are multiple ways to provide credentials:

- Configuring Nodes to Authenticate to a Private Registry (requires node configuration by cluster administrator)
- Pre-pulled Images
- Specifying ImagePullSecrets on a Pod
- etc.

??? example "Example - imagePullSecrets"
    Assuming that the `myregistrykey` [Secret](https://kubernetes.io/docs/concepts/configuration/secret/#docker-config-secrets) has been already created in the cluster, which contains credentials for accessing the container registry, we can use the the `imagePullSecrets` field in the Pod spec, for example:

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-image-pull-policy-and-secrets
      labels:
        app.kubernetes.io/name: nginx-image-pull-policy-and-secrets
    spec:
      replicas: 1
      selector:
        matchLabels:
          app.kubernetes.io/name: nginx-image-pull-policy-and-secrets
      template:
        metadata:
          labels:
            app.kubernetes.io/name: nginx-image-pull-policy-and-secrets
        spec:
          containers:
          - name: nginx
            image: nginx:1.23
            imagePullPolicy: IfNotPresent
          imagePullSecrets:
          - name: myregistrykey
    ```


## ImagePullBackOff

Sometimes your Pods might get an `ImagePullBackOff` status, which means that a container could not start because Kubernetes could not pull a container image for reasons such as:

- an invalid image name, 
- pulling from a private registry without an `imagePullSecret` or using an incorrect `imagePullSecret`, etc. 

The `BackOff` part indicates that Kubernetes will keep trying to pull the image, with an increasing back-off delay.

Kubernetes raises the delay between each attempt until it reaches a compiled-in limit, which is 300 seconds (5 minutes).


## Garbage collection of unused images

The kubelet performs garbage collection on unused images every five minutes.

To configure options for unused image garbage collection, tune the kubelet using a [configuration file](https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/).
