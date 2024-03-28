# Kubectl Commands

The following sections provide some usuful `kubectl` commands when working with Pod resources.

## Viewing and finding resources

A few kubectl commands to view and find Pod resources:

| Command | Description |
|:-------:|:-----------:|
| `kubectl get pods --all-namespaces` | List all pods in all namespaces |
| `kubectl get pods -o wide` | List all pods in the current namespace, with more details |
| `kubectl get pods` | List all pods in the namespace |
| `kubectl get pod my-pod -o yaml` | Get a pod's YAML |
| `kubectl describe pods my-pod` | Show details of a specific Pod |
| `kubectl get pods --sort-by='.status.containerStatuses[0].restartCount'` | List pods Sorted by Restart Count |
| `kubectl get pods --selector=app=cassandra -o jsonpath='{.items[*].metadata.labels.version}'` | Get the version label of all pods with label `app=cassandra` |
| `kubectl get pods --field-selector=status.phase=Running` | Get all running pods in the namespace |


## Interacting with Pods

A few kubectl commands to interact with running Pods:

| Command | Description |
|:-------:|:-----------:|
| `kubectl logs my-pod` | dump pod logs (stdout) |
| `kubectl logs -l name=myLabel` | dump pod logs, with label name=myLabel (stdout) |
| `kubectl logs my-pod --previous` | dump pod logs (stdout) for a previous instantiation of a container |
| `kubectl logs my-pod -c my-container` | dump pod container logs (stdout, multi-container case) |
| `kubectl logs -l name=myLabel -c my-container` | dump pod logs, with label name=myLabel (stdout) |
| `kubectl logs my-pod -c my-container --previous` | dump pod container logs (stdout, multi-container case) for a previous instantiation of a container |
| `kubectl logs -f my-pod` | stream pod logs (stdout) |
| `kubectl logs -f my-pod -c my-container` | stream pod container logs (stdout, multi-container case) |
| `kubectl logs -f -l name=myLabel --all-containers` | stream all pods logs with label name=myLabel (stdout) |
| `kubectl run -i --tty busybox --image=busybox:1.28 -- sh` | Run pod as interactive shell |
| `kubectl run nginx --image=nginx -n mynamespace` |Start a single instance of nginx pod in the namespace of mynamespace |
| `kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml` | Generate spec for running pod nginx and write it into a file called pod.yaml |
| `kubectl port-forward my-pod 5000:6000` | Listen on port 5000 on the local machine and forward to port 6000 on my-pod |
| `kubectl exec my-pod -- ls /` | Run command in existing pod (1 container case) |
| `kubectl exec --stdin --tty my-pod -- /bin/sh` | Interactive shell access to a running pod (1 container case) |
| `kubectl exec my-pod -c my-container -- ls /` | Run command in existing pod (multi-container case) |
| `kubectl top pod POD_NAME --containers` | Show metrics for a given pod and its containers |
| `kubectl top pod POD_NAME --sort-by=cpu` | Show metrics for a given pod and sort it by 'cpu' or 'memory' |


## Updating resources

A few kubectl commands to update Pod resources:

| Command | Description |
|:-------:|:-----------:|
| `kubectl label pods my-pod new-label=awesome` | Add the label `new-label=awesome` to the Pod named `my-pod` |
| `kubectl label pods my-pod new-label-` | Remove the label `new-label` from the Pod named `my-pod` |
| `kubectl annotate pods my-pod web-url=http://cern.ch` | Add an annotation |

!!! tip
    When updating or manipulating resources, the declarative approach should be prefered over editing on the fly with `kubectl` commands.
