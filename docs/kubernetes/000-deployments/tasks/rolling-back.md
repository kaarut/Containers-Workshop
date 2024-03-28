# Rolling back a Deployment

Sometimes, you may want to rollback a Deployment; for example, when the Deployment is not stable, such as crash looping. By default, all of the Deployment's rollout history is kept in the system so that you can rollback anytime you want (you can change that by modifying revision history limit).

!!! note
    A Deployment's revision is created when a Deployment's rollout is triggered. This means that the new revision is created if and only if the Deployment's Pod template (`.spec.template`) is changed, for example if you update the labels or container images of the template. Other updates, such as scaling the Deployment, do not create a Deployment revision, so that you can facilitate simultaneous manual- or auto-scaling. This means that when you roll back to an earlier revision, only the Deployment's Pod template part is rolled back.


## Prerequisites

The following prerequisites should be met:

- Have the nginx Deployment object applied to your cluster, as described in the initial section about Deployments.


## Example

### Updating Deployment with a misconfiguration

- Suppose that you made a typo while updating the Deployment, by putting the image name as `nginx:1.161` instead of `nginx:1.16.1`:

    ```bash
    kubectl set image deployment/nginx-deployment nginx=nginx:1.161
    ```

- The rollout gets stuck. You can verify it by checking the rollout status:

    ```bash
    kubectl rollout status deployment/nginx-deployment
    ```

    The output is similar to this:

    ```bash
    Waiting for rollout to finish: 1 out of 3 new replicas have been updated...
    ```

    Looking at the Pods created, you see that 1 Pod created by new ReplicaSet is stuck in an image pull loop:

    ```bash
    $ kubectl get pods

    NAME                               READY   STATUS             RESTARTS   AGE
    nginx-deployment-c7865cb47-mkcbw   1/1     Running            0          104m
    nginx-deployment-c7865cb47-n2pf7   1/1     Running            0          104m
    nginx-deployment-c7865cb47-wljdg   1/1     Running            0          104m
    nginx-deployment-df4d5858f-hjmtd   0/1     ImagePullBackOff   0          22s
    ```

    To fix this, you need to rollback to a previous revision of Deployment that is stable.

    !!! note
        The Deployment controller stops the bad rollout automatically, and stops scaling up the new ReplicaSet. This depends on the rollingUpdate parameters (`maxUnavailable` specifically) that you have specified. Kubernetes by default sets the value to 25%.

### Checking Rollout History of a Deployment

1. Check the revisions of this Deployment:

    ```bash
    kubectl rollout history deployment/nginx-deployment
    ```

1. To see the details of each revision, run:

    ```bash
    kubectl rollout history deployment/nginx-deployment --revision=2
    ```

    The output is similar to this:

    ```bash
    deployments "nginx-deployment" revision 2
    Labels:       app=nginx
            pod-template-hash=1159050644
    Annotations:  kubernetes.io/change-cause=kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
    Containers:
    nginx:
        Image:      nginx:1.16.1
        Port:       80/TCP
        QoS Tier:
            cpu:      BestEffort
            memory:   BestEffort
        Environment Variables:      <none>
    No volumes.
    ```

### Rolling Back to a Previous Revision

1. First, check the revisions of this Deployment:

    ```bash
    kubectl rollout history deployment/nginx-deployment
    ```
    The output is similar to this:

    ```bash
    deployments "nginx-deployment"
    REVISION    CHANGE-CAUSE
    1           kubectl apply --filename=https://k8s.io/examples/controllers/nginx-deployment.yaml
    2           kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
    3           kubectl set image deployment/nginx-deployment nginx=nginx:1.161
    ```

    !!! note
        If `CHANGE-CAUSE` values are empty, it's because `CHANGE-CAUSE` is copied from the Deployment annotation `kubernetes.io/change-cause` to its revisions upon creation. You can specify the `CHANGE-CAUSE` message by:

        - Annotating the Deployment with `kubectl annotate deployment/nginx-deployment kubernetes.io/change-cause="image updated to 1.16.1"`
        - Manually editing the manifest of the resource.

1. To see the details of each revision, run:

    ```bash
    kubectl rollout history deployment/nginx-deployment --revision=2
    ```

    The output is similar to this:

    ```bash
    deployments "nginx-deployment" revision 2
    Labels:       app=nginx
            pod-template-hash=1159050644
    Annotations:  kubernetes.io/change-cause=kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
    Containers:
    nginx:
        Image:      nginx:1.16.1
        Port:       80/TCP
        QoS Tier:
            cpu:      BestEffort
            memory:   BestEffort
        Environment Variables:      <none>
    No volumes.
    ```

### Rolling Back to a Previous Revision

Follow the steps given below to rollback the Deployment from the current version to the previous version, which is version 2.

1. Now you've decided to undo the current rollout and rollback to the previous revision:

    ```bash
    kubectl rollout undo deployment/nginx-deployment
    ```

    The output is similar to this:

    ```bash
    deployment.apps/nginx-deployment rolled back
    ```

    _Alternatively_, you can rollback to a specific revision by specifying it with --to-revision:

    ```bash
    kubectl rollout undo deployment/nginx-deployment --to-revision=2
    ```

    The output is similar to this:

    ```bash
    deployment.apps/nginx-deployment rolled back
    ```

    The Deployment is now rolled back to a previous stable revision. As you can see, a `DeploymentRollback` event for rolling back to revision 2 is generated from Deployment controller.

1. Check if the rollback was successful and the Deployment is running as expected, run:

    ```bash
    kubectl get deployment nginx-deployment
    ```

    The output is similar to this:

    ```bash
    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   3/3     3            3           30m
    ```

1. Get the description of the Deployment:
    ```bash
    kubectl describe deployment nginx-deployment
    ```
