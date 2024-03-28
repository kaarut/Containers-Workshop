# ConfigMaps

**A ConfigMap is an API object used to store _non-confidential_ data in key-value pairs**. Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume.

!!! danger "Warning"
    **ConfigMap does not provide secrecy or encryption**. If the data you want to store are confidential, use a Secret rather than a ConfigMap, or use additional (third party) tools to keep your data private.

!!! note
    **A ConfigMap is not designed to hold large chunks of data**. The data stored in a ConfigMap cannot exceed 1 MiB. If you need to store settings that are larger than this limit, you may want to consider mounting a volume or use a separate database or file service.


## ConfigMap object

- Unlike most Kubernetes objects that have a `spec`, a ConfigMap has `data` and `binaryData` fields.
    - The `data` field is designed to contain UTF-8 strings while the `binaryData` field is designed to contain binary data as base64-encoded strings.
- The name of a ConfigMap must be a [valid DNS subdomain name](../000-objects/names-and-ids.md#names).
- Each key under the `data` or the `binaryData` field must consist of alphanumeric characters, `-`, `_` or `.`. The keys stored in `data` must not overlap with the keys in the `binaryData` field.


## ConfigMaps and Pods

You can write a Pod spec that refers to a ConfigMap and configures the container(s) in that Pod based on the data in the ConfigMap. The Pod and the ConfigMap must be in the same namespace.

### Example

#### ConfigMap creation

Here's an example ConfigMap that has some keys with single values, and other keys where the value looks like a fragment of a configuration format:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
data:
  # property-like keys; each key maps to a simple value
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"

  # file-like keys
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true
```

To create this ConfigMap in our Kubernetes cluster use the `kubectl apply` command.


!!! info
    You can also use `kubectl create configmap` to create a ConfigMap from an individual file, or from multiple files.

#### Pod creation

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-configmap-demo
spec:
  containers:
  - name: nginx
    image: nginx
    env:
    # Define the environment variable
    - name: PLAYER_INITIAL_LIVES
      valueFrom:
        configMapKeyRef:
          name: game-demo           # The ConfigMap this value comes from.
          key: player_initial_lives # The key to fetch.
    - name: UI_PROPERTIES_FILE_NAME
      valueFrom:
        configMapKeyRef:
          name: game-demo
          key: ui_properties_file_name
    volumeMounts:
    - name: config
      mountPath: "/config"
      readOnly: true
  volumes:
  # You set volumes at the Pod level, then mount them into containers inside that Pod
  - name: config
    configMap:
      # Provide the name of the ConfigMap you want to mount.
      name: game-demo
      # An array of keys from the ConfigMap to create as files
      items:
      - key: "game.properties"
        path: "game.properties"
      - key: "user-interface.properties"
        path: "user-interface.properties"
```

In the above example, defining a volume and mounting it inside the `nginx` container as `/config` creates two files, `/config/game.properties` and `/config/user-interface.properties`, even though there are four keys in the ConfigMap. This is because the Pod definition specifies an `items` array in the `volumes` section. If you omit the `items` array entirely, every key in the ConfigMap becomes a file with the same name as the key, and you get 4 files.


## Immutable ConfigMaps

Kubernetes provides an option to make ConfigMaps immutable. Some use-cases:

- protect you from accidental (or unwanted) updates that could cause applications outages
- improve performance of your cluster by significantly reducing load on kube-apiserver, by closing watches for ConfigMaps marked as immutable.

You can create an immutable ConfigMap by setting the `immutable` field to `true`. For example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  ...
data:
  ...
immutable: true
```

!!! note
    Once a ConfigMap is marked as immutable, it is _not_ possible to revert this change nor to mutate the contents of the `data` field. You can only delete and recreate the ConfigMap.
