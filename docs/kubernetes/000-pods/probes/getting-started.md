# Container Probes

A _probe_ is a diagnostic performed periodically by the `kubelet` on a container. To perform a diagnostic, the kubelet either executes code within the container, or makes a network request.

Probes are used to detect:

- Containers that haven’t started yet and can’t serve traffic.
- Containers that are overwhelmed and can’t serve additional traffic.
- Containers that are completely dead and not serving any traffic.


## Check mechanisms

There are currently four different ways to check a container using a probe. Each probe must define exactly one of these four mechanisms:

| Value | Description |
|:-----:|:-----------:|
| `exec` | Executes a specified command inside the container. The diagnostic is considered successful if the command exits with a status code of 0. It is useful when you don't run an HTTP server, but can run a command that can check whether or not your app is healthy. |
| `grpc` | Performs a remote procedure call using [gRPC](https://grpc.io/). The target should implement [gRPC health checks](https://grpc.github.io/grpc/core/md_doc_health-checking.html). The diagnostic is considered successful if the `status` of the response is `SERVING`. gRPC probes are an alpha feature and are only available if you enable the `GRPCContainerProbe` feature gate. |
| `httpGet` | Performs an HTTP `GET` request against the Pod's IP address on a specified port and path. The diagnostic is considered successful if the response has a status code greater than or equal to 200 and less than 400. |
| `tcpSocket` | Performs a TCP check against the Pod's IP address on a specified port. The diagnostic is considered successful if the port is open. If the remote system (the container) closes the connection immediately after it opens, this counts as healthy. |

## Probe outcome

Each probe has one of three results:

| Value | Description |
|:-----:|:-----------:|
| `Success` | The container passed the diagnostic. |
| `Failure` | The container failed the diagnostic. |
| `Unknown` | The diagnostic failed (no action should be taken, and the kubelet will make further checks). |

## Types of probe

The kubelet can optionally perform and react to three kinds of probes on running containers:

| Value | Description |
|:-----:|:-----------:|
| `livenessProbe` | Indicates whether the container is running. If the liveness probe fails, the kubelet kills the container, and the container is subjected to its restart policy. If a container does not provide a liveness probe, the default state is `Success`. |
| `readinessProbe` | Indicates whether the container is ready to respond to requests. If the readiness probe fails, the endpoints controller removes the Pod's IP address from the endpoints of all Services that match the Pod. The default state of readiness before the initial delay is `Failure`. If a container does not provide a readiness probe, the default state is `Success`. |
| `startupProbe` | Indicates whether the application within the container is started. All other probes are disabled if a startup probe is provided, until it succeeds. If the startup probe fails, the kubelet kills the container, and the container is subjected to its restart policy. If a container does not provide a startup probe, the default state is `Success`. |


### When should you use a liveness probe?

You should use a liveness probe if you'd like your container to be killed and restarted (it should be combined with a `restartPolicy` of `Always` or `OnFailure`).

### When should you use a readiness probe?

**If you'd like to start sending traffic to a Pod only when a probe succeeds, specify a readiness probe**. In this case, the readiness probe might be the same as the liveness probe, but the existence of the readiness probe in the spec means that the Pod will start without receiving any traffic and only start receiving traffic after the probe starts succeeding.

If you want your container to be able to take itself down for maintenance, you can specify a readiness probe that checks an endpoint specific to readiness that is different from the liveness probe.

If your app has a strict dependency on back-end services, you can implement both a liveness and a readiness probe. **The liveness probe passes when the app itself is healthy, but the readiness probe additionally checks that each required back-end service is available**. This helps you avoid directing traffic to Pods that can only respond with error messages.

### When should you use a startup probe?

**Startup probes are useful for Pods that have containers that take a long time to come into service**. Rather than set a long liveness interval, you can configure a separate configuration for probing the container as it starts up, allowing a time longer than the liveness interval would allow.


## Common Error Scenarios

### Cascading Failures

A readiness probe response can be conditional on components that are outside the direct control of the application. For example, you could configure a readiness probe using `HTTPGet`, in such a way that the application first checks the availability of a cache service or database before responding to the probe. This means that if the database is down or late to respond, the entire application will become unavailable.

This may or may not make sense, depending on your application setup. If the application cannot function at all without the third-party component, maybe this behavior is warranted. If it can continue functioning, for example, by falling back to a local cache, the database or external cache should not be connected to probe responses.

### Delayed Response

In some circumstances, readiness probes may be late to respond—for example, if the application needs to read large amounts of data with low latency or perform heavy computations. Consider this behavior when configuring readiness probes, and always test your application thoroughly before running it in production with a readiness probe.


Kubernetes uses liveness probes to know when to restart a container. If a container is unresponsive—perhaps the application is deadlocked due to a multi-threading defect—restarting the container can make the application more available, despite the defect. It certainly beats paging someone in the middle of the night to restart a container.
