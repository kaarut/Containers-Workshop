# Configure Liveness, Readiness and Startup Probes

## Overview

Kubernetes provides probes (health checks) to monitor and act on the state of Pods (Containers) and to make sure only healthy Pods serve traffic. With help of Probes, we can control when a pod should be deemed started, ready for service, or live to serve traffic.

## Kubelet and Probes

How `kubelet` uses probes:

- The `kubelet` uses **liveness probes** to know when to restart a container. For example, liveness probes could catch a deadlock, where an application is running, but unable to make progress. Restarting a container in such a state can help to make the application more available despite bugs.

- The `kubelet` uses **readiness probes** to know when a container is ready to start accepting traffic. A Pod is considered ready when all of its containers are ready. One use of this signal is to control which Pods are used as backends for Services. When a Pod is not ready, it is removed from Service load balancers.

- The `kubelet` uses **startup probes** to know when a container application has started. If such a probe is configured, it disables liveness and readiness checks until it succeeds, making sure those probes don't interfere with the application startup. This can be used to adopt liveness checks on slow starting containers, avoiding them getting killed by the kubelet before they are up and running.

## Configuration Options

Probes have a number of fields that you can use to more precisely control the behavior of startup, liveness and readiness checks:
 
| Value | Description |
|:-----:|:-----------:|
| `initialDelaySeconds` | Number of seconds after the container has started before startup, liveness or readiness probes are initiated. |
| `periodSeconds` | How often (in seconds) to perform the probe. |
| `timeoutSeconds` | Number of seconds after which the probe times out. |
| `successThreshold` | Minimum consecutive successes for the probe to be considered successful after having failed. |
| `failureThreshold` | When a probe fails, Kubernetes will try `failureThreshold` times before giving up. Giving up in case of liveness probe means restarting the container. In case of readiness probe the Pod will be marked Unready. |

!!! info
    For the default values of each configuration option please consult the [official Kubernetes documentation](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#configure-probes).

## HTTP Probes
HTTP probes have additional fields that can be set on `httpGet`:

- `host`
- `scheme`
- `path`
- `httpHeaders`
- `port`

For an HTTP probe, the kubelet sends an HTTP request to the specified path and port to perform the check. 

## Define Liveness command

Let's create a Pod that runs a single `busybox` container:

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

In the configuration file, you can see that the Pod:

- has a single Container
- the `periodSeconds` field specifies that the kubelet should perform a liveness probe every 5 seconds
- the `initialDelaySeconds` field tells the kubelet that it should wait 5 seconds before performing the first probe

To perform a probe, the kubelet executes the command `cat /tmp/healthy` in the target container. If the command succeeds, it returns `0`, and the kubelet considers the container to be alive and healthy. If the command returns a non-zero value, the kubelet kills the container and restarts it.

_When the container starts_, it executes this command:

```bash
/bin/sh -c "touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600"
```

For the first 30 seconds of the container's life, there is a /tmp/healthy file. So during the first 30 seconds, the command cat /tmp/healthy returns a success code. After 30 seconds, cat /tmp/healthy returns a failure code.

_Within 30 seconds_, view the Pod events:

```bash
Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  11s   default-scheduler  Successfully assigned default/liveness-exec to node01
  Normal  Pulling    9s    kubelet, node01    Pulling image "registry.k8s.io/busybox"
  Normal  Pulled     7s    kubelet, node01    Successfully pulled image "registry.k8s.io/busybox"
  Normal  Created    7s    kubelet, node01    Created container liveness
  Normal  Started    7s    kubelet, node01    Started container liveness
```

_After 35 seconds_, view the Pod events again:

```bash
kubectl describe pod liveness-exec
```

At the bottom of the output, there are messages indicating that the liveness probes have failed, and the failed containers have been killed and recreated.

```bash
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  57s                default-scheduler  Successfully assigned default/liveness-exec to node01
  Normal   Pulling    55s                kubelet, node01    Pulling image "registry.k8s.io/busybox"
  Normal   Pulled     53s                kubelet, node01    Successfully pulled image "registry.k8s.io/busybox"
  Normal   Created    53s                kubelet, node01    Created container liveness
  Normal   Started    53s                kubelet, node01    Started container liveness
  Warning  Unhealthy  10s (x3 over 20s)  kubelet, node01    Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
  Normal   Killing    10s                kubelet, node01    Container liveness failed liveness probe, will be restarted
```

_Wait another 30 seconds_, and verify that the container has been restarted:

```bash
kubectl get pod liveness-exec
```

The output shows that `RESTARTS` has been incremented. Note that the `RESTARTS` counter increments as soon as a failed container comes back to the running state:

```bash
NAME            READY     STATUS    RESTARTS   AGE
liveness-exec   1/1       Running   1          1m
```

## Define a liveness HTTP request

Another kind of liveness probe uses an HTTP GET request. Here is the configuration file for a Pod that runs a container with an HTTP server:

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/liveness
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
```

In the configuration file, you can see that the Pod has a single container. The `periodSeconds` field specifies that the kubelet should perform a liveness probe every 3 seconds. The `initialDelaySeconds` field tells the kubelet that it should wait 3 seconds before performing the first probe. To perform a probe, the kubelet sends an HTTP GET request to the server that is running in the container and listening on port `8080`. If the handler for the server's `/healthz` path returns a success code, the kubelet considers the container to be alive and healthy. If the handler returns a failure code, the kubelet kills the container and restarts it.

Any code greater than or equal to 200 and less than 400 indicates success. Any other code indicates failure.

You can see the source code for the server in [`server.go`](https://github.com/kubernetes/kubernetes/blob/master/test/images/agnhost/liveness/server.go).

For the first 10 seconds that the container is alive, the `/healthz` handler returns a status of 200. After that, the handler returns a status of 500.

```go
http.HandleFunc("/healthz", func(w http.ResponseWriter, r *http.Request) {
    duration := time.Now().Sub(started)
    if duration.Seconds() > 10 {
        w.WriteHeader(500)
        w.Write([]byte(fmt.Sprintf("error: %v", duration.Seconds())))
    } else {
        w.WriteHeader(200)
        w.Write([]byte("ok"))
    }
})
```

The kubelet starts performing health checks 3 seconds after the container starts. So the first couple of health checks will succeed. But after 10 seconds, the health checks will fail, and the kubelet will kill and restart the container.

_After 10 seconds since the creation of the Pod_, view Pod events to verify that liveness probes have failed and the container has been restarted:

```bash
kubectl describe pod liveness-http
```

and view the Pod events at the bottom of the output:

```bash
Events:
  Type     Reason     Age              From               Message
  ----     ------     ----             ----               -------
  Normal   Scheduled  15s              default-scheduler  Successfully assigned default/liveness-http to cms-daq-workshop-gml7jxg5oxyf-node-0
  Normal   Pulling    15s              kubelet            Pulling image "registry.k8s.io/liveness"
  Normal   Pulled     14s              kubelet            Successfully pulled image "registry.k8s.io/liveness" in 1.168412846s
  Normal   Created    14s              kubelet            Created container liveness
  Normal   Started    14s              kubelet            Started container liveness
  Warning  Unhealthy  1s (x2 over 4s)  kubelet            Liveness probe failed: HTTP probe failed with statuscode: 500
```

## Use a named port

You can use a named port for HTTP and TCP probes. (gRPC probes do not support named ports).

For example:

```yaml
ports:
- name: liveness-port
  containerPort: 8080
  hostPort: 8080

livenessProbe:
  httpGet:
    path: /healthz
    port: liveness-port
```

## Protect slow starting containers with startup probes

Sometimes, you have to deal with legacy applications that might require an additional startup time on their first initialization. In such cases, it can be tricky to set up liveness probe parameters without compromising the fast response to deadlocks that motivated such a probe. The trick is to set up a startup probe with the same command, HTTP or TCP check, with a `failureThreshold * periodSeconds` long enough to cover the worse case startup time.

So, the previous example would become:

```yaml
ports:
- name: liveness-port
  containerPort: 8080
  hostPort: 8080

livenessProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 1
  periodSeconds: 10

startupProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 30
  periodSeconds: 10
```

Thanks to the startup probe, the application will have a maximum of 5 minutes (30 * 10 = 300s) to finish its startup. Once the startup probe has succeeded once, the liveness probe takes over to provide a fast response to container deadlocks. If the startup probe never succeeds, the container is killed after 300s and subject to the pod's `restartPolicy`.
