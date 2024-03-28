# Copy files/folders

Even though the `docker cp` is not directly related to storage, it might be useful for debugging purposes and interact with the host-container filesystem.

!!!note
    `docker cp` is quite useful for debugging and development purposes. **The declarative approach should be prefered in most cases**.


## Usage

```bash
docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
```

The `docker cp` utility copies the contents of SRC_PATH to the DEST_PATH. You can copy from the container’s file system to the local machine or the reverse, from the local filesystem to the container.

## Examples

### Copy a local file into container

```bash
docker cp ./some_file CONTAINER:/work
```

### Copy files from container to local path

```bash
docker cp CONTAINER:/var/logs/ /tmp/app_logs
```

## References

- [`docker cp` official docs](https://docs.docker.com/engine/reference/commandline/cp/)
