# Using subPath

Sometimes, it is useful to share one volume for multiple uses in a single pod. The `volumeMounts.subPath` property specifies a sub-path inside the referenced volume instead of its root.


## Problem Statement

When working with volumes, we often encounter some problems like:

- As we mount a Kubernetes volume at a mount-point and if we already have some content in it, then it would be **hidden by our mount**. But in some cases, we would not want to hide the existing content at the mount-point but want to **add additional files/directories in parallel**.
- While working with **multiple ConfigMaps/Secrets**, if one wants to **mount multiple keys from different ConfigMaps/Secrets at the same or other location** but cannot do it as it **gets overwritten**. 


To solve such types of problems, a `subPath` in volume comes to the rescue.


## Location on host

As we have already mentioned in a previous section, the path of volume on the host is created under the following path `/var/lib/kubelet/pods/<POD_UID>/volumes/<VOLUME_TYPE>/<VOLUME_NAME>`.

For example, have a look at the following snippet of a Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis
    volumeMounts:
    - name: redis-storage
      mountPath: /data/redis
  volumes:
  - name: redis-storage
    emptyDir: {}
```

If we're **not using `subPath` (only `mountPath`)** then at the host, volume is created at `/var/lib/kubelet/pods/<POD_UID>/volumes/kubernetes.io~empty-dir/redis-storage`.

On the other hand, if we’re **using subPath** as `dataset1` then at the host, its path looks like `/var/lib/kubelet/pods/<POD_UID>/volume-subpaths/volume1`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis-volume-subpath
spec:
  containers:
  - name: redis
    image: redis
    volumeMounts:
    - mountPath: /mnt/data/dataset1
      name: volume1
      subPath: dataset1
  volumes:
  - name: volume1
    emptyDir: {}
```

Therefore, **`subPath` will append the data to the existing volume mount point by creating a new directory and will not overwrite the existing content which makes the same volume to be mounted multiple times**.

This is mostly useful **when you want to mount a configuration file from a ConfigMap or want to mount credentials from a Secret but not to mount it as a volume**.


## Example with ConfigMap

Let's illustrate some of the subPath properties by using (and then mounting) a ConfigMap into a Pod.

### ConfigMap creation

Let's create our ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-subpath-example
data:
  hello.html: |
    <!--hello.html-->
    <html>
    <title>subPath</title>
    <body>
    <h1>Hello, This is a subPath demo</h1>
    </body>
    </html>
```

### Deployment without subPath

Let's create our Deployment object without using `subPath` for the volumeMount:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-no-subpath
spec:
  containers:
  - image: nginx
    name: nginx
    volumeMounts:
    - name: nginx-volume
      mountPath: /usr/share/nginx/html/
  volumes:
  - name: nginx-volume
    configMap:
      name: nginx-subpath-example
```


This takes all the map names and data sources of the ConfigMap named `nginx-subpath-example` and mounts it as a volume at `/usr/share/nginx/html/`:

```bash
$ kubectl exec -it nginx-no-subpath -- ls /usr/share/nginx/html/

hello.html
```

The volume mount worked. Kubernetes took the map name of `hello.html` and present it as a file with the contents that were stored in the data source of the ConfigMap. The problem however is it laid that volume on top of the existing directory. The default files for Nginx are no longer present. You'd have to create all the Nginx files and store them into the ConfigMap. Or, we can use a `subPath`.


### Deployment with subPath

Let's create our Deployment object using `subPath` for the volumeMount:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-with-subpath
spec:
  containers:
  - image: nginx
    name: nginx
    volumeMounts:
    - name: nginx-volume
      mountPath: /usr/share/nginx/html/hello.html
      subPath: hello.html
  volumes:
  - name: nginx-volume
    configMap:
      name: nginx-subpath-example
      items:
      - key: hello.html
        path: hello.html
```

Some of the changes needed:

- update `volumeMounts.mountPath` to include the file name I want it to mount. 
- add the `volumeMounts.subPath` property. The value for the `subPath` must match the path specified in `volumes` section.
- update the `volumes` section. Instead of simply providing the ConfigMap name, we now also need to provide the items list of the entries we want to include from the ConfigMap. Under items, we specify the key, which is the map name and the path. The `path` value must match the `subPath` value defined in `.spec.volumeMounts`.

Let's view the contents of the static sites in our Nginx container:

```bash
$ kubectl exec -it nginx-with-subpath -- ls /usr/share/nginx/html/

50x.html  hello.html  index.html
```

As you can see, the default Nginx HTML files (i.e. `50x.html` and `index.html`) are also present.


## Downsides

**SubPaths are not automatically updated when a ConfigMap is modified**. Changes to a ConfigMap will need to be a new deployment which would result in the pods being recreated with the updated ConfigMap content.
