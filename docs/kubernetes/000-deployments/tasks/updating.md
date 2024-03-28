# Updating a Deployment

## Prerequisites

The following prerequisites should be met:

- Have the nginx Deployment object applied to your cluster, as described in the initial chapter about Deployments.

## Example

1. Let's update the nginx Pods to use the `nginx:1.16.1` image instead of the `nginx:1.14.2` image:

    ```bash
    kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
    ```

    The output is similar to:

    ```bash
    deployment.apps/nginx-deployment image updated
    ```

    _Alternatively_, you can edit the Deployment and change `.spec.template.spec.containers[0].image` from `nginx:1.14.2` to `nginx:1.16.1`:

    ```bash
    kubectl edit deployment/nginx-deployment
    ```

1. To see the rollout status, run:

    ```bash
    kubectl rollout status deployment/nginx-deployment
    ```

    The output is similar to this:

    ```bash
    Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
    ```

    _or_

    ```bash
    deployment "nginx-deployment" successfully rolled out
    ```

    !!! note
        A Deployment's rollout is triggered if and only if the Deployment's Pod template (that is, `.spec.template`) is changed, for example if the labels or container images of the template are updated. Other updates, such as scaling the Deployment, do not trigger a rollout.

1. Get more details on your updated Deployment:

    - After the rollout succeeds, you can view the Deployment by running:
        
        ```bash
        kubectl get deployments
        ```
        
        The output is similar to this:

        ```bash
        NAME               READY   UP-TO-DATE   AVAILABLE   AGE
        nginx-deployment   3/3     3            3           26h
        ```

    - Run `kubectl get rs` to see that the Deployment updated the Pods by creating a new ReplicaSet and scaling it up to 3 replicas, as well as scaling down the old ReplicaSet to 0 replicas.

        ```bash
        kubectl get rs
        ```

        The output is similar to this:

        ```bash
        NAME                          DESIRED   CURRENT   READY   AGE
        nginx-deployment-66b6c48dd5   0         0         0       26h
        nginx-deployment-c7865cb47    3         3         3       103m
        ```

        !!! note
            A Deployment's revision history is stored in the ReplicaSets it controls.

            The `.spec.revisionHistoryLimit` field is an optional field that specifies the number of old ReplicaSets to retain to allow rollback. These old ReplicaSets consume resources in `etcd` and crowd the output of `kubectl get rs`. The configuration of each Deployment revision is stored in its ReplicaSets; therefore, once an old ReplicaSet is deleted, you lose the ability to rollback to that revision of Deployment. By default, 10 old ReplicaSets will be kept.

    - Running get pods should now show only the new Pods:


        ```bash
        kubectl get pods
        ```
        
        The output is similar to this:

        ```bash
        NAME                                READY     STATUS    RESTARTS   AGE
        nginx-deployment-c7865cb47-mkcbw   1/1     Running   0          91m
        nginx-deployment-c7865cb47-n2pf7   1/1     Running   0          91m
        nginx-deployment-c7865cb47-wljdg   1/1     Running   0          91m
        ```

        Next time you want to update these Pods, you only need to update the Deployment's Pod template again.

        Deployment ensures that only a certain number of Pods are down while they are being updated. By default, it ensures that at least 75% of the desired number of Pods are up (25% max unavailable).

        Deployment also ensures that only a certain number of Pods are created above the desired number of Pods. By default, it ensures that at most 125% of the desired number of Pods are up (25% max surge).

    - Get details of your Deployment:

        ```bash
        kubectl describe deployments
        ```
