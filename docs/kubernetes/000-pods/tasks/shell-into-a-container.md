# Get a Shell to a Running Container

In this section we're going to have a look at `kubectl exec` command to get a shell to a running container.


## Example

- Let's create an nginx Pod:

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx-shell-demo
    spec:
      containers:
      - name: nginx
        image: nginx
    ```


- Verify that the container is running:

    ```bash
    kubectl get pods nginx-shell-demo
    ```

- Get a shell to the running container:

    ```bash
    kubectl exec -it nginx-shell-demo -- /bin/bash
    ```

    !!! note
        The double dash (`--`) separates the arguments you want to pass to the command from the kubectl arguments.

- In your shell (inside the container), you can experiment with running a few commands:

    ```bash
    ls -al /  # List files and directories in the root dir
    whoami    # Display the username of the current user

    apt-get update -y          # Update the package lists

    apt-get install -y curl    # Install curl
    curl http://localhost      # Access nginx's webpage

    apt-get install -y procps  # Install the 'procps' package
    ps aux                     # Display current processes
    ```

- In an ordinary command window (not the shell of the container), you can run such commands directly on the running container:

    ```bash
    kubectl exec -it nginx-shell-demo -- ls -al /
    kubectl exec -it nginx-shell-demo -- env
    ```

## Opening a shell when a Pod has more than one container

If a Pod has more than one container, use `--container` or `-c` to specify a container in the `kubectl exec` command.

For example, let's have a look at a Pod with two containers:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-two-containers
spec:

  volumes:
  - name: shared-data
    emptyDir: {}

  containers:

  - name: nginx
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html

  - name: alpine
    image: alpine
    volumeMounts:
    - name: shared-data
      mountPath: /pod-data
    command: ["/bin/sh"]
    args: ["-c", "while true; do wget -O /pod-data/index.html http://info.cern.ch; sleep 30; done"]
```

To open a shell to the `alpine` container:

```bash
kubectl exec -it pod-with-two-containers -c alpine -- /bin/sh
```
