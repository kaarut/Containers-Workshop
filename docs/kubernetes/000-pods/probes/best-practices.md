# Probes Best Practices

Kubernetes uses **readiness** probes to decide when the container is available for accepting traffic. The readiness probe is used to control which pods are used as the backends for a service. A pod is considered ready when all of its containers are ready. If a pod is not ready, it is removed from service load balancers. For example, if a container loads a large cache at startup and takes minutes to start, you do not want to send requests to this container until it is ready, or the requests will fail—you want to route requests to other pods, which are capable of servicing requests.

Kubernetes uses **liveness** probes to know when to restart a container. If a container is unresponsive—perhaps the application is deadlocked due to a multi-threading defect—restarting the container can make the application more available, despite the defect. It certainly beats paging someone in the middle of the night to restart a container.

!!! info
    In concurrent computing, deadlock is any situation in which no member of some group of entities can proceed because each waits for another member, including itself, to take action, such as sending a message or, more commonly, releasing a lock.

## Readiness Probe

We want to avoid routing requests to the pod until it is ready to accept traffic. However, the readiness probe will continue to be called throughout the lifetime of the container, every `periodSeconds`, so that the container can make itself temporarily unavailable when one of its dependencies is unavailable, or while running a large batch job, performing maintenance, or something similar.

The recommended way to implement the Readiness probe is for your application to expose a `/readyz` HTTP endpoint. When it receives a request on this endpoint, the application should send a `200 OK` response if it is ready to receive traffic. Ready to receive traffic means the following:

- Application is healthy.
- Any potential initialization steps are completed.
- Any valid request sent to the application doesn’t result in an error.


## Liveness Probe

Recall that a liveness-probe failure will result in the container being restarted. Unlike a readiness probe, it is not idiomatic to check dependencies in a liveness probe. **A liveness probe should be used to check if the container itself has become unresponsive**.

The recommended way to implement the Liveness probe is for application to expose a `/livez` HTTP endpoint. When receiving a request on this endpoint, the application should send a `200 OK` response if it is considered healthy.

## Resources

- [Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
- [Kubernetes Liveness and Readiness Probes: How to Avoid Shooting Yourself in the Foot](https://blog.colinbreck.com/kubernetes-liveness-and-readiness-probes-how-to-avoid-shooting-yourself-in-the-foot/)
