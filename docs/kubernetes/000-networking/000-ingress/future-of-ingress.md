# The Future of Ingress

As you have seen, the Ingress object provides a very useful abstraction for configuring L7 load balancers⁠—but it hasn’t scaled to all the features that users want and various implementations are looking to offer. Many of the features in Ingress are underdefined. Implementations can surface these features in different ways, reducing the portability of configurations between implementations.

Another problem is that **Ingress is easy to misconfigure Ingress**. The way that multiple objects combine opens the door for conflicts that are resolved differently by different implementations. In addition, the way that these are merged across namespaces breaks the idea of namespace isolation.

Ingress was also created before the idea of a [service mesh](../000-extras/service-mesh/getting-started.md) was well known. The intersection of Ingress and service meshes is still being defined.

**The future of HTTP load balancing for Kubernetes looks to be the [Gateway API](https://gateway-api.sigs.k8s.io/)**, which is in the midst of development by the Kubernetes special interest group (SIG) dedicated to networking. The Gateway API project is intended to develop a **more modern API for routing in Kubernetes**. Though it is more focused on HTTP balancing, Gateway also includes resources for controlling Layer 4 (TCP) balancing. The Gateway APIs are still very much under development, so it is strongly recommended that people stick to the existing Ingress and Service resources that are currently present in Kubernetes. The current state of the Gateway API can be found online.
