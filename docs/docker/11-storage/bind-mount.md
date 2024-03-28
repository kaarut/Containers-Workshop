# Bind Mounts

When you use a bind mount, a file or directory on the host machine is mounted into a container. Bind mounts have limited functionality compared to volumes. The file or directory is referenced by its absolute path on the host machine. By contrast, when you use a volume, a new directory is created within Docker’s storage directory on the host machine, and Docker manages that directory’s contents.

![Bind mounts](./bind-mount-types-of-mounts.png)


## Start a Container with a bind mount

Consider a case where you have a directory `source` and that when you build the source code, the artifacts are saved into another directory, `source/target/`. You want the artifacts to be available to the container at `/app/`, and you want the container to get access to a new build each time you build the source on your development host.

Below is an _example_ with files in the Docker host machine:

```bash
$ (~/Desktop/temp-docker-bind-test) tree

.
├── my_code
└── target
    ├── bind-test-file-1
    └── bind-test-file-2

1 directory, 3 files
```

Use the following command to bind-mount the `target/` directory into your container at `/app/`. Run the command from within the `source` directory. The `$(pwd)` sub-command expands to the current working directory on Linux or macOS hosts.


```bash
docker run -d \
    -it \
    --name cms-daq-bind-mount-test \
    --mount type=bind,source="$(pwd)"/target,target=/app \
    nginx:latest
```

!!!tip
    For bind mounts, both the `--mount` and the `--volume` (or `-v`) flags can be used.

    New users should use the `--mount` syntax. Experienced users may be more familiar with the `-v` or `--volume` syntax, but are encouraged to use `--mount`.


Use `docker inspect cms-daq-bind-mount-test` to verify that the bind mount was created correctly:


```bash
$ docker inspect cms-daq-bind-mount-test --format '{{json .Mounts}}'

[
  {
    "Type": "bind",
    "Source": "/root/temp-docker-bind-test/target",
    "Destination": "/app",
    "Mode": "",
    "RW": true,
    "Propagation": "rprivate"
  }
]
```

This shows that the mount is a `bind` mount, it shows the correct source and destination, it shows that the mount is read-write, and that the propagation is set to `rprivate`.

To confirm that the files from your filesystem were mounted inside the container under `/app/`, run the following command:

```bash
$ docker exec -it cms-daq-bind-mount-test ls -al /app/

total 0
drwxr-xr-x. 2 root root 54 Nov 28 11:15 .
drwxr-xr-x. 1 root root 50 Nov 28 11:16 ..
-rw-r--r--. 1 root root  0 Nov 28 11:15 bind-test-file-1
-rw-r--r--. 1 root root  0 Nov 28 11:15 bind-test-file-2
```

To stop and remove the container:

```bash
$ docker container stop cms-daq-bind-mount-test

$ docker container rm cms-daq-bind-mount-test
```
