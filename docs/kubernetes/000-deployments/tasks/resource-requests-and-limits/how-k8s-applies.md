# How Kubernetes applies resource requests and limits

When the kubelet starts a container as part of a Pod, the kubelet passes that container's requests and limits for memory and CPU to the container runtime.

On Linux, the container runtime typically configures **kernel cgroups** that apply and enforce the limits you defined.

- The **CPU limit** defines a hard ceiling on how much CPU time that the container can use. During each scheduling interval (time slice), the Linux kernel checks to see if this limit is exceeded; if so, the kernel waits before allowing that cgroup to resume execution.
- The **CPU request** typically defines a weighting. If several different containers (cgroups) want to run on a contended system, workloads with larger CPU requests are allocated more CPU time than workloads with small requests.
- The **memory request** is mainly used during (Kubernetes) Pod scheduling. On a node that uses cgroups v2, the container runtime might use the memory request as a hint to set `memory.min` and `memory.low`.
- The **memory limit** defines a memory limit for that cgroup. If the container tries to allocate more memory than this limit, the Linux kernel out-of-memory subsystem activates and, typically, intervenes by stopping one of the processes in the container that tried to allocate memory. If that process is the container's PID 1, and the container is marked as restartable, Kubernetes restarts the container.


If a container exceeds its memory request and the node that it runs on becomes short of memory overall, it is likely that the Pod the container belongs to will be evicted.

A container might or might not be allowed to exceed its CPU limit for extended periods of time. However, container runtimes don't terminate Pods or containers for excessive CPU usage.


## Cgroups

To illustrate how containers and Pods resource requests and limits use Linux's kernet cgroups, let's have a closer look at an example. We're going to use the Deployment object found at the beginning of this section (i.e. the one that contains two containers with resource requests and limits).

1. Get the name of one of running Pods:

    ```bash
    kubectl get pods -l app.kubernetes.io/name=resource-req-and-limits
    ```

1. Get the worker node name of this Pod. We could use the `kubectl describe` command:

    ```bash
    kubectl describe pod resource-req-and-limits-564dbc654d-cj5db
    ```

    The output should be similar to this:

    ```bash
    Name:             resource-req-and-limits-564dbc654d-cj5db
    Namespace:        default
    Node:             cms-daq-workshop-gml7jxg5oxyf-node-1/188.185.124.119
    Labels:           app.kubernetes.io/name=resource-req-and-limits
    ...
    ```

1. SSH to this worker node (make sure to replace `<IP_ADDRESS>` with your worker node's IP address):

    ```bash
    ssh core@<IP_ADDRESS>
    ```

1. Get the PID of your running Redis container on the worker node:

    ```bash
    REDIS_PID=$(ps aux | grep -i redis | grep -v grep | awk '{print $2}')
    ```

    (Make sure that the `REDIS_PID` variable is not empty: `echo $REDIS_PID`.)

1. Get information about this PID's cgroup:

    ```bash
    cat /proc/$REDIS_PID/cgroup
    ```

    The output should look like this:

    ```bash
    ...

    8:cpu,cpuacct:/kubepods/burstable/podeb1f6abc-474e-45ca-ad02-fad2ac1cfc3f/f007a7ab664298c097876eadcf7dcf04a7bf8ea18dd1340dfceaef8a6d1482e9
    7:memory:/kubepods/burstable/podeb1f6abc-474e-45ca-ad02-fad2ac1cfc3f/f007a7ab664298c097876eadcf7dcf04a7bf8ea18dd1340dfceaef8a6d1482e9

    ...
    ```

1. View this cgroup's maximum amount of user memory:

    ```bash
    cat /sys/fs/cgroup/memory/kubepods/burstable/podeb1f6abc-474e-45ca-ad02-fad2ac1cfc3f/f007a7ab664298c097876eadcf7dcf04a7bf8ea18dd1340dfceaef8a6d1482e9/memory.limit_in_bytes
    ```

    !!! note
        The `/kubepods/burstable/podeb1f6abc-474e-45ca-ad02-fad2ac1cfc3f/f007a7ab664298c097876eadcf7dcf04a7bf8ea18dd1340dfceaef8a6d1482e9` part is what is returned from the previous command. Make sure to replace it accordingly.

    The output of this command is the memory limits in bytes. In this case it's `629145600` bytes, which translates to `600` Megabytes (hint: it's calculated with the following formula: `Bytes / 1,048,576`, as a Megabyte has 1,048,576 Bytes). This value (i.e. 600Mi) matches the value we defined in the Deployment object for the resource memory limits on the Redis container.

1. View this cgroup's CPU shares:

    ```bash
    cat /sys/fs/cgroup/cpu/kubepods/burstable/podeb1f6abc-474e-45ca-ad02-fad2ac1cfc3f/f007a7ab664298c097876eadcf7dcf04a7bf8ea18dd1340dfceaef8a6d1482e9/cpu.shares
    ```

    !!! info
        The `cpu.shares` contains an integer value that specifies a relative share of CPU time available to the tasks in a cgroup. For example, tasks in two cgroups that have `cpu.shares` set to `100` will receive equal CPU time, but tasks in a cgroup that has `cpu.shares` set to `200` receive twice the CPU time of tasks in a cgroup where cpu.shares is set to `100`. The value specified in the `cpu.shares` file must be `2` or higher.

    The output of this command is the CPU shares. In this case it's `512`, which matches the Redis container's CPU limit value we defined when creating our Deployment object.

    !!! note
        Why 512, and not 500? The cpu control group divides a core into 1024 shares, whereas Kubernetes divides it into 1000.

## Reads

- [RHEL docs, Resource Management Guide, CPU](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/resource_management_guide/sec-cpu)
- [RHEL docs, Resource Management Guide, Memory](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/resource_management_guide/sec-memory)
- [Blog Post, Understanding resource limits in kubernetes: cpu time](https://medium.com/@betz.mark/understanding-resource-limits-in-kubernetes-cpu-time-9eff74d3161b)
- [Blog Post, Demystifying Kubernetes CPU Limits (and Throttling)](https://wbhegedus.me/understanding-kubernetes-cpu-limits/)
