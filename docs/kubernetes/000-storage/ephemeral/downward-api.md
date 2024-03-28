# Downward API Volume

This page shows how a Pod can use a downwardAPI volume, to expose information about itself to containers running in the Pod. A `downwardAPI` volume can expose Pod fields and container fields.

In Kubernetes, there are two ways to expose Pod **and** container fields to a running container:

- Environment variables
- Volume files

## Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubernetes-downwardapi-volume-example
  labels:
    zone: us-est-coast
    cluster: test-cluster1
    rack: rack-22
  annotations:
    build: two
    builder: john-doe
spec:
  containers:
    - name: client-container
      image: registry.k8s.io/busybox
      command: ["sh", "-c"]
      args:
      - while true; do
          echo -en '\n\n### Printing Pod-related file contents that were mounted as Downward API volumes..';
          if [[ -e /etc/podinfo/labels ]]; then
            echo -en '\n'; cat /etc/podinfo/labels; fi;
          if [[ -e /etc/podinfo/annotations ]]; then
            echo -en '\n\n'; cat /etc/podinfo/annotations; fi;
          echo -en '\n\n### Printing Container-related file contents that were mounted as Downward API volumes..';
          if [[ -e /etc/podinfo/cpu_limit ]]; then
            echo -en '\nCPU Limit:'; cat /etc/podinfo/cpu_limit; fi;
          if [[ -e /etc/podinfo/cpu_request ]]; then
            echo -en '\nCPU Request:'; cat /etc/podinfo/cpu_request; fi;
          if [[ -e /etc/podinfo/mem_limit ]]; then
            echo -en '\nMemory Limit:'; cat /etc/podinfo/mem_limit; fi;
          if [[ -e /etc/podinfo/mem_request ]]; then
            echo -en '\nMemory Request:'; cat /etc/podinfo/mem_request; fi;
          sleep 5;
        done;
      resources:
        requests:
          memory: "32Mi"
          cpu: "125m"
        limits:
          memory: "64Mi"
          cpu: "250m"
      volumeMounts:
        - name: podinfo
          mountPath: /etc/podinfo
  volumes:
    - name: podinfo
      downwardAPI:
        items:
          # Pod-related fields as volumes from the Downward API
          - path: "labels"
            fieldRef:
              fieldPath: metadata.labels
          - path: "annotations"
            fieldRef:
              fieldPath: metadata.annotations
          # Container-related fields as volumes from the Downward API
          - path: "cpu_limit"
            resourceFieldRef:
              containerName: client-container
              resource: limits.cpu
              divisor: 1m
          - path: "cpu_request"
            resourceFieldRef:
              containerName: client-container
              resource: requests.cpu
              divisor: 1m
          - path: "mem_limit"
            resourceFieldRef:
              containerName: client-container
              resource: limits.memory
              divisor: 1Mi
          - path: "mem_request"
            resourceFieldRef:
              containerName: client-container
              resource: requests.memory
              divisor: 1Mi
```

In the manifest, you can see that the Pod has a `downwardAPI` volume, and that the single container in that Pod mounts the volume at `/etc/podinfo`.

Each element of the `.spec.volumes[0].downwardAPI.items` array defines a file in the downward API volume.

View the container's logs and confirm you get back the expected results:

```bash
kubectl logs kubernetes-downwardapi-volume-example
```

You can also get a shell into the container that is running in your Pod and view the contents of the mounted files using the Downward API:

```bash
kubectl exec -it kubernetes-downwardapi-volume-example -- sh
```

and then interact with the Downward API volume:

```bash
ls -al /etc/podinfo/
```
