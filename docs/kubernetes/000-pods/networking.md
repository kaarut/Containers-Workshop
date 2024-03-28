# Pod Networking

!!! info
    This is a _short_ introduction to Pod networking. More details can be found on the Networking chapter of this workshop (or in the official Kubernetes documentation).

Some key points:

- Each Pod is assigned a unique IP address for each address family.
- Every container in a Pod shares the network namespace, including the IP address and network ports.
    - Inside a Pod (and only then), the containers that belong to the Pod can communicate with one another using `localhost`.
- When containers in a Pod communicate with entities outside the Pod, they must coordinate how they use the shared network resources (such as ports).
- The containers in a Pod can also communicate with each other using standard inter-process communications like SystemV semaphores or POSIX shared memory.
- Containers that want to interact with a container running in a different Pod can use IP networking to communicate.
- Containers within the Pod see the system hostname as being the same as the configured name for the Pod.
