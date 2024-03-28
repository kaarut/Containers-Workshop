# Scaling a Deployment

## Prerequisites

The following prerequisites should be met:

- Have the nginx Deployment object applied to your cluster, as described in the initial chapter about Deployments.

## Example

- You can scale a Deployment by using the following command:

    ```bash
    $ kubectl scale deployment/nginx-deployment --replicas=10


    deployment.apps/nginx-deployment scaled
    ```

- If you list all Pods, 10 replicas of the nginx Deployment should appear:

    ```bash
    $ kubectl get pods


    NAME                                READY   STATUS    RESTARTS   AGE
    nginx-deployment-559d658b74-2t9hn   1/1     Running   0          6m30s
    nginx-deployment-559d658b74-8k9lm   1/1     Running   0          21s
    nginx-deployment-559d658b74-bwlvj   1/1     Running   0          6m32s
    nginx-deployment-559d658b74-jxllm   1/1     Running   0          21s
    nginx-deployment-559d658b74-mdrdg   1/1     Running   0          21s
    nginx-deployment-559d658b74-nl2f5   1/1     Running   0          6m29s
    nginx-deployment-559d658b74-s8758   1/1     Running   0          21s
    nginx-deployment-559d658b74-wjhhh   1/1     Running   0          21s
    nginx-deployment-559d658b74-x4tft   1/1     Running   0          21s
    nginx-deployment-559d658b74-zrj8j   1/1     Running   0          21s
    ```


!!! info
    If [horizontal Pod autoscaling (HPA)](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) is enabled in your cluster, you can set up an autoscaler for your Deployment and choose the minimum and maximum number of Pods you want to run based on the CPU utilization of your existing Pods.

    More on HPA later in this workshop.
