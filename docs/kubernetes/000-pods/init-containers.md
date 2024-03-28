# Init Containers

**Init containers are specialized containers that run before app containers in a Pod**. Init containers can contain utilities or setup scripts not present in an app image.

## Example format

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod

spec:

  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', "MY_INIT_COMMAND"]

  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'MY_CONTAINER_COMMAND']
```

## Understanding Init Containers

A Pod can have multiple containers running apps within it, but it can also have one or more init containers, which are run before the app containers are started.

Init containers are exactly like regular containers, except:

- **Init containers always run to completion**.
- **Each init container must complete successfully before the next one starts**.

If a Pod's init container fails, the kubelet repeatedly restarts that init container until it succeeds. However, if the Pod has a `restartPolicy` of Never, and an init container fails during startup of that Pod, Kubernetes treats the overall Pod as failed.

## Differences from regular containers

Init containers support all the fields and features of app containers, including resource limits, volumes, and security settings. However, the resource requests and limits for an init container are handled differently, as documented [here](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#resources).

Also, init containers do not support lifecycle, livenessProbe, readinessProbe, or startupProbe because they must run to completion before the Pod can be ready.

If you specify multiple init containers for a Pod, kubelet runs each init container sequentially. Each init container must succeed before the next can run. When all of the init containers have run to completion, kubelet initializes the application containers for the Pod and runs them as usual.

## Using Init Containers

### Advantages for start-up related code

- Because init containers run to completion before any app containers start, **init containers offer a mechanism to block or delay app container startup until a set of preconditions are met**. Once preconditions are met, all of the app containers in a Pod can start in parallel.
- Init containers can securely run utilities or custom code that would otherwise make an app container image less secure. By keeping unnecessary tools separate you can limit the attack surface of your app container image.

### Use cases

- Wait for a Service to be created, using a shell one-line command like:
    ```bash
    for i in {1..100}; do sleep 1; if dig myservice; then exit 0; fi; done; exit 1
    ```
- Clone a Git repository into a Volume
- Wait for some time before starting the app container (e.g. using the `sleep` command)

## Detailed behavior

During Pod startup, the kubelet delays running init containers until the networking and storage are ready. Then the kubelet runs the Pod's init containers in the order they appear in the Pod's spec.

Each init container must exit successfully before the next container starts. If a container fails to start due to the runtime or exits with failure, it is retried according to the Pod `restartPolicy`. However, if the Pod `restartPolicy` is set to Always, the init containers use `restartPolicy` OnFailure.

A Pod cannot be `Ready` until all init containers have succeeded. The ports on an init container are not aggregated under a Service. A Pod that is initializing is in the `Pending` state but should have a condition `Initialized` set to false.

If the Pod restarts, or is restarted, all init containers must execute again.

Because init containers can be restarted, retried, or re-executed, init container code should be idempotent. In particular, code that writes to files on EmptyDirs should be prepared for the possibility that an output file already exists.
