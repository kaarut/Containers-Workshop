# Restart Policy

Kubernetes provides this capability that can automatically restart containers whenever they fail.

Restart policies are an important component of self-healing applications, which are automatically repaired when a problem arises.

## Always

Create a Pod definition with `restartPolicy` set to `Always`:

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: always-pod
spec:
    restartPolicy: Always
    containers:
    - name: busybox
      image: busybox
      command: ['sh', '-c', 'sleep 10']
```

Once the above manifest is applied to the Kubernetes cluster, the Pod was created, exited after 10 seconds and it was restarted by the kubelet and moved to a `Running` status.

```diff
$ kubectl get pods

NAME                              READY   STATUS    RESTARTS   AGE
+ always-pod                      1/1     Running   0          5s
nginx-cms-daq-workshop            1/1     Running   0          6d20h
nginx-cms-daq-workshop-multiple   2/2     Running   0          5d23h



$ kubectl get pods

NAME                              READY   STATUS    RESTARTS     AGE
+ always-pod                      1/1     Running   1 (7s ago)   18s
nginx-cms-daq-workshop            1/1     Running   0            6d20h
nginx-cms-daq-workshop-multiple   2/2     Running   0            5d23h
```

## OnFailure

Create a Pod definition with `restartPolicy` set to `OnFailure`:

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: onfailure-pod
spec:
    restartPolicy: OnFailure
    containers:
    - name: busybox
      image: busybox
      command: ['sh', '-c', 'sleep 10']
```

Once the above manifest is applied to the Kubernetes cluster, you can observe the initial Pod creation and that the Pod is not restarted after completion, as it exited with a zero status code:

```diff
$ kubectl get pods

NAME                              READY   STATUS    RESTARTS        AGE
always-pod                        1/1     Running   6 (2m56s ago)   6m52s
nginx-cms-daq-workshop            1/1     Running   0               6d20h
nginx-cms-daq-workshop-multiple   2/2     Running   0               6d
+ onfailure-pod                   1/1     Running   0               8s



$ kubectl get pods

NAME                              READY   STATUS             RESTARTS      AGE
always-pod                        0/1     CrashLoopBackOff   6 (65s ago)   8m
nginx-cms-daq-workshop            1/1     Running            0             6d20h
nginx-cms-daq-workshop-multiple   2/2     Running            0             6d
+ onfailure-pod                   0/1     Completed          0             76s
```

!!! example
    **Exercise**: Create a Pod object with `restartPolicy: OnFailure` which will exit with an error after a while:

    ```yaml
    command: ['sh', '-c', 'sleep 10 && echo "Exiting $(date)" && exit 1']
    ```

## Never

Create a Pod definition with `restartPolicy` set to `Never`:

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: never-pod
spec:
    restartPolicy: Never
    containers:
    - name: busybox
      image: busybox
      command: ['sh', '-c', 'sleep 10']
```

Once the above manifest is applied to the Kubernetes cluster, you can observe the initial Pod creation and that the Pod is never restarted after completion.

```diff
$ kubectl get pods

NAME                              READY   STATUS             RESTARTS         AGE
always-pod                        0/1     CrashLoopBackOff   34 (4m11s ago)   159m
+ never-pod                       1/1     Running            0                6s
nginx-cms-daq-workshop            1/1     Running            0                6d22h
nginx-cms-daq-workshop-multiple   2/2     Running            0                6d2h
onfailure-pod                     0/1     Completed          0                153m


$ kubectl get pods

NAME                              READY   STATUS             RESTARTS         AGE
always-pod                        0/1     CrashLoopBackOff   34 (4m25s ago)   160m
+ never-pod                       0/1     Completed          0                20s
nginx-cms-daq-workshop            1/1     Running            0                6d22h
nginx-cms-daq-workshop-multiple   2/2     Running            0                6d2h
onfailure-pod                     0/1     Completed          0                153m
```
