# Object Names and IDs

Each object in your cluster has a [Name](#names) that is unique for that type of resource. Every Kubernetes object also has a [UID](#uids) that is unique across your whole cluster.

For example, you can only have one Pod named `myapp-1234` within the same namespace, but you can have one Pod and one Deployment that are each named `myapp-1234`.

## Names

Only one object of a given kind can have a given name at a time.

Below are some types of commonly used name constraints for resources:

- [RFC 1123](https://tools.ietf.org/html/rfc1123) subdomain names - e.g. contain no more than 253 characters
- [RFC 1123](https://tools.ietf.org/html/rfc1123) DNS Label standards - e.g. start with an alphanumeric character, contain only lowercase alphanumeric characters or '-', etc.
- [RFC 1035](https://www.rfc-editor.org/rfc/rfc1035) DNS Label standards - e.g. contain at most 63 characters, start with an alphabetic character, etc.


More details can be found on the [official Kubernetes documentation about object names](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names).

## UIDs

A Kubernetes systems-generated string to uniquely identify objects.

Every object created over the whole lifetime of a Kubernetes cluster has a distinct UID. It is intended to distinguish between historical occurrences of similar entities.

Kubernetes UIDs are universally unique identifiers (also known as UUIDs).
