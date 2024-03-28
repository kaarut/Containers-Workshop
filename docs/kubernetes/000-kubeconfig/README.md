# Kubeconfig

kubeconfig files organize information about clusters, users, namespaces, and authentication mechanisms.

Then, the `kubectl` command-line tool uses kubeconfig files to find the information it needs to choose a cluster and communicate with the API server of a cluster.

!!! info
    _kubeconfig_ file is a file that is used to configure access to Kubernetes clusters.

!!! warning
    Only use kubeconfig files from trusted sources. Using a specially-crafted kubeconfig file could result in malicious code execution or file exposure.

## File location

By default, `kubectl` looks for a file named `config` in the `$HOME/.kube` directory. You can specify other kubeconfig files by setting the `KUBECONFIG` environment variable or by setting the `--kubeconfig` flag.

## Context

A _context_ element in a kubeconfig file is used to group access parameters under a convenient name. Each context has three parameters: cluster, namespace, and user. By default, the `kubectl` command-line tool uses parameters from the current context to communicate with the cluster.

To choose the current context:

```bash
kubectl config use-context
```

## File Example

```yaml
apiVersion: v1

clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://1.2.3.4:6443
  name: cms-daq-workshop

contexts:
- context:
    cluster: cms-daq-workshop
    user: admin
  name: default

current-context: default
kind: Config
preferences: {}

users:
- name: admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```

## OpenStack Magnum cluster Configuration

If you have successfully:

- installed the OpenStack and Magnum clients
- installed `kubectl` client
- created your Kubernetes on OpenStack

you should be able to fetch the kubeconfig configuration for the created Kubernetes cluster:

```bash
CLUSTER_NAME="cms-daq-workshop" && \
mkdir -p $HOME/cern/openstack/$CLUSTER_NAME && \
openstack coe cluster config $CLUSTER_NAME --dir $HOME/cern/openstack/$CLUSTER_NAME
```

Set the `KUBECONFIG` environment variable:

```bash
export KUBECONFIG=$HOME/cern/openstack/$CLUSTER_NAME/config
```

You can preview the contents this kubeconfig file:

```bash
less $HOME/cern/openstack/$CLUSTER_NAME/config
```

!!! warning
    **Do not share** the kubeconfig file with anyone, otherwise someone will get access to your cluster and might run malicious workloads.
