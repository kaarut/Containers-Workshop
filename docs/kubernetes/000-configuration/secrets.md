# Secrets

A Secret is an **object that contains a small amount of sensitive data** such as a password, a token, or a key.

Such information might otherwise be put in a Pod specification or in a container image. Using a Secret means that you don't need to include confidential data in your application code.

Secrets give you **more control over how sensitive information is used and reduces the risk of accidental exposure**.

!!! note
    Secrets are similar to [ConfigMaps](./configmaps.md) but are specifically intended to hold confidential data.


!!! danger
    **Kubernetes Secrets are, by default, stored unencrypted** in the API server's underlying data store (etcd).
    
    Anyone with API access can retrieve or modify a Secret, and so can anyone with access to etcd.
    
    Additionally, **anyone who is authorized to create a Pod in a namespace can use that access to read any Secret in that namespace**; this includes indirect access such as the ability to create a Deployment.


## Uses for Secrets

There are three main ways for a Pod to use a Secret:

- As files in a volume mounted on one or more of its containers.
- As container environment variable.
- By the kubelet when pulling images for the Pod.


## Creating a Secret

There are several options to create a Secret:

- Use kubectl
- Use a configuration file
- Use the Kustomize tool

### Using kubectl

#### Raw data

```bash
kubectl create secret generic db-user-pass \
    --from-literal=username=admin \
    --from-literal=password='S!B\*d$zDsb='
```

!!! note
    You must use single quotes `''` to escape special characters such as `$`, `\`, `*`, `=`, and `!` in your strings. If you don't, your shell will interpret these characters.

#### Source files

1. Store the credentials in files:

    ```bash
    echo -n 'admin' > ./username.txt
    echo -n 'S!B\*d$zDsb=' > ./password.txt
    ```

    !!! info
        The `-n` flag ensures that the generated files do not have an extra newline character at the end of the text.

    You do not need to escape special characters in strings that you include in a file.

1. Pass the file paths in the `kubectl` command:

    ```bash
    kubectl create secret generic db-user-pass \
        --from-file=./username.txt \
        --from-file=./password.txt
    ```

    The default key name is the file name. You can optionally set the key name using `--from-file=[key=]source`. For example:
    
    ```bash
    kubectl create secret generic db-user-pass \
        --from-file=username=./username.txt \
        --from-file=password=./password.txt
    ```

#### Configuration files

You can specify the `data` and/or the `stringData` field when creating a configuration file for a Secret. The `data` and the `stringData` fields are optional. The values for all keys in the data field have to be base64-encoded strings. If the conversion to base64 string is not desirable, you can choose to specify the `stringData` field instead, which accepts arbitrary strings as values.


## Using a Secret

Secrets can be mounted as data volumes or exposed as environment variables to be used by a container in a Pod.

**A Secret needs to be created before any Pods that depend on it**.

### Volumes

Let's create a Pod with a signle container that uses volumes to mount sensitive data from a Secret:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-secret-volume
spec:
  volumes:
  - name: secret-volume
    secret:
      secretName: db-user-pass
      # defaultMode: 0400
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: secret-volume
      readOnly: true
      mountPath: "/etc/secret-volume"
```

Now use `kubectl exec` command to exec into the container of the Pod we created and confirm that the secret data have been mounted on `/etc/secret-volume`:

```bash
kubectl exec -it nginx-secret-volume -- sh -c 'ls -al /etc/secret-volume/'
```

When a volume contains data from a Secret, and that Secret is updated, Kubernetes tracks this and updates the data in the volume, using an eventually-consistent approach.


### Define a container environment variable with data from a single Secret

You can consume the data in Secrets as environment variables in your containers.

**If a container already consumes a Secret in an environment variable, a Secret update will not be seen by the container unless it is restarted**. There are third party solutions for triggering restarts when secrets change.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-single-secret
spec:
  containers:
  - name: envars-test-container
    image: nginx
    env:
    - name: SECRET_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-user-pass
          key: password.txt
```


## Size limit

Individual secrets are **limited to 1MiB** in size. This is to discourage creation of very large secrets that could exhaust the API server and kubelet memory. However, creation of many smaller secrets could also exhaust memory. You can use a resource quota to limit the number of Secrets (or other resources) in a namespace. For example:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: object-counts
spec:
  hard:
    configmaps: "10"
    pods: "4"
    secrets: "10"
    services: "10"
```

More details can be found on the [official Kubernetes documentation for Resource Quotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/).


## Retrieving a Secret

To retrieve a Secret you can use the `kubectl get` command.

1. View the contents of the Secret you created:

    ```bash
    kubectl get secret db-user-pass -o jsonpath='{.data}'
    ```

1. To decode the password data:

    ```bash
    kubectl get secret db-user-pass -o jsonpath='{.data.password\.txt}' | base64 -d
    ```

!!! note
    The commands `kubectl get` and `kubectl describe` avoid showing the contents of a `Secret` by default. This is to protect the `Secret` from being exposed accidentally, or from being stored in a terminal log.


## Types of Secret


When creating a Secret, you can specify its type using the `type` field.

The Secret type is used to facilitate programmatic handling of the Secret data.

Some of the built-in Secret types that Kubernetes provides:

| Built-in Type | Usage |
|:-------------:|:-----:|
| `Opaque` | arbitrary user-defined data |
| `kubernetes.io/service-account-token` | ServiceAccount token |
| `kubernetes.io/dockercfg` | serialized ~/.dockercfg file |
| `kubernetes.io/dockerconfigjson` | serialized ~/.docker/config.json file |
| `kubernetes.io/basic-auth` | credentials for basic authentication |
| `kubernetes.io/ssh-auth` | credentials for SSH authentication |
| `kubernetes.io/tls` | data for a TLS client or server |
| `bootstrap.kubernetes.io/token` | bootstrap token data |


### Opaque secrets

`Opaque` is the **default Secret type** if omitted from a Secret configuration file. When you create a Secret using `kubectl`, you will use the `generic` subcommand to indicate an `Opaque` Secret type. For example, the following command creates an empty Secret of type `Opaque`:

```bash
kubectl create secret generic empty-secret

kubectl get secret empty-secret
```

### Service account token Secrets

A `kubernetes.io/service-account-token` type of Secret is used to store a token credential that identifies a service account.

### Docker config Secrets

You can use one of the following `type` values to create a Secret to store the credentials for **accessing a container image registry**:

- `kubernetes.io/dockercfg` (legacy format)
- `kubernetes.io/dockerconfigjson` (new format)

The `kubernetes.io/dockerconfigjson` type is designed for storing a serialized JSON that follows the same format rules as the `~/.docker/config.json` file.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-dockercfg
type: kubernetes.io/dockercfg
data:
  .dockercfg: |
    "<base64 encoded ~/.dockercfg file>"  
```

### TLS secrets

Kubernetes provides a builtin Secret type `kubernetes.io/tls` for storing a certificate and its associated key that are typically used for TLS.

One common use for TLS secrets is to configure encryption in transit for an Ingress, but you can also use it with other resources or directly in your workload. When using this type of Secret, the `tls.key` and the `tls.crt` key must be provided in the `data`:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-tls
type: kubernetes.io/tls
data:
  # the data is abbreviated in this example
  tls.crt: |
    MIIC2DCCAcCgAwIBAgIBATANBgkqh ...    
  tls.key: |
    MIIEpgIBAAKCAQEA7yn3bRHQ5FHMQ ...    
```


## Immutable Secrets

Kubernetes lets you mark specific Secrets as _immutable_.

You can create an immutable Secret by setting the `immutable` field to `true`. For example,

```yaml
apiVersion: v1
kind: Secret
metadata:
  ...
data:
  ...
immutable: true
```

!!! note
    Once a Secret is marked as immutable, it is not possible to revert this change nor to mutate the contents of the `data` field. You can only delete and recreate the Secret.


## Information security for Secrets 

Although ConfigMap and Secret work similarly, Kubernetes applies some additional protection for Secret objects.

A Secret is only sent to a node if a Pod on that node requires it. For mounting secrets into Pods, the kubelet stores a copy of the data into a `tmpfs` so that the confidential data is not written to durable storage. Once the Pod that depends on the Secret is deleted, the kubelet deletes its local copy of the confidential data from the Secret.

!!! info
    tmpfs (short for Temporary File System) is a temporary file storage paradigm implemented in many Unix-like operating systems. It is intended to appear as a mounted file system, but data is stored in volatile memory instead of a persistent storage device.

!!! danger
    Any containers that run with `privileged: true` on a node can access all Secrets used on that node.


## Encryption on etcd

As we mentioned above, Kubernetes Secrets are, by default, stored unencrypted in the API server's underlying data store (etcd) and anyone with API access (or etcd access) can retrieve or modify a Secret.

In order to safely use Secrets, take at least the following steps:

- [Enable Encryption at Rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/) for Secrets.
- [Enable or configure RBAC rules](https://kubernetes.io/docs/reference/access-authn-authz/authorization/) with least-privilege access to Secrets.
    - Restrict `get`, `watch`, or `list` access to Secrets. Only allow cluster administrators to access `etcd`.
    - Restrict `create` verb for Pods. A user who can create a Pod that uses a Secret can also see the value of that Secret (by using the `kubectl exec` command).
- Restrict Secret access to specific containers.
    - If you are defining multiple containers in a Pod, and only one of those containers needs access to a Secret, define the volume mount or environment variable configuration so that the other containers do not have access to that Secret.
- Consider using [external Secret store providers](https://secrets-store-csi-driver.sigs.k8s.io/concepts.html#provider-for-the-secrets-store-csi-driver):


    ![Vault flow - Example](./img/secrts/vault-flow-example.avif)
