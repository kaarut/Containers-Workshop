# Kubectl Commands

The following sections provide some usuful `kubectl` commands when working with Deployment resources.

## Viewing and finding resources

A few kubectl commands to view and find Pod resources:

| Command | Description |
|:-------:|:-----------:|
| `kubectl get deployment nginx-deployment` | List a particular deployment |
| `kubectl describe deployment nginx-deployment` | Show details of a specific Deployment |


## Updating resources

A few kubectl commands to view and update Pod resources:

| Command | Description |
|:-------:|:-----------:|
| `kubectl set image deployment/frontend www=image:v2` | Rolling update `www` containers of `frontend` deployment, updating the image |
| `kubectl rollout history deployment/nginx-deployment` | Check the history of deployments including the revision |
| `kubectl rollout undo deployment/nginx-deployment` | Rollback to the previous deployment |
| `kubectl rollout undo deployment/nginx-deployment --to-revision=2` | Rollback to a specific revision |
| `kubectl rollout status -w deployment/nginx-deployment` | Watch rolling update status of `nginx-deployment` Deployment until completion |
| `kubectl rollout restart deployment/nginx-deployment` | Rolling restart of the `nginx-deployment` Deployment |


!!! tip
    When updating or manipulating resources, the declarative approach should be prefered over editing on the fly with `kubectl` commands.


## Interacting with Deployments

A few kubectl commands to interact Pod resources:

| Command | Description |
|:-------:|:-----------:|
| `kubectl logs deploy/nginx-deployment` | dump Pod logs for a Deployment (single-container case) |
| `kubectl logs deploy/my-deployment -c my-container` | dump Pod logs for a Deployment (multi-container case) |
| `kubectl port-forward deploy/nginx-deployment 5000:80` | listen on local port 5000 and forward to port 80 on a Pod created by `nginx-deployment` |
| `kubectl exec deploy/nginx-deployment -- ls` | run command in first Pod and first container in Deployment (single- or multi-container cases) |
