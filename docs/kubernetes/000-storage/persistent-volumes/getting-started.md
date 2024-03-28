# Persistent volumes - Getting started

Containers were developed to be stateless, ephemeral, lightweight tools, only megabytes in size, to speed application launch. However, this design is problematic when data needs to persist after the container goes away.

**To ensure that data persists well beyond a container’s lifecycle, the best practice is to separate data management from containers**. There are three approaches to data persistence in a container environment:

- Storage plugins
- Data volume containers
- Building a local directory mount into the container as a data directory


## Storage Considerations

1. Storage that doesn't depend on the pod lifecycle.
    - The storage will still be there, even if the pod dies and a new one is being created. Therefore, when the new can pick up where the previous one left off so that it will read the existing data from that storage to get up-to-date data.
1. Storage must be available on all nodes.
1. Storage needs to survive even if the cluster crashes.


## Local vs Remote volume types

The Local volume types violate two of the storage considerations of data persistence:

- local volumes are tied to one specific node. This means that a pod either has to be scheduled on a specific node every time, otherwise the data will not be available if a pod is being scheduled in a different node.
- local volumes don't survive cluster crashes.
