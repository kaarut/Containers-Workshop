# Build Arguments

The `ARG` instruction is used to define build arguments in the Dockerfile.

Usage:

```docker
ARG <name>[=<default value>]
```

The `ARG` instruction defines a variable that users can pass at build-time to the builder with the `docker build` command using the `--build-arg <varname>=<value>` flag. If a user specifies a build argument that was not defined in the Dockerfile, the build outputs a warning:

```
[Warning] One or more build-args [foo] were not consumed.
```

!!! warning
    It is not recommended to use build-time variables for passing secrets like GitHub keys, user credentials etc. Build-time variable values are visible to any user of the image with the d`ocker history` command.


```docker
FROM busybox

# Build argument with no default value. The value of this argument
# has to be provided when building the image.
ARG user1

# Build argument with a default value
ARG buildno=1

# ...
```

`ARG` is the only instruction that may precede `FROM` in the `Dockerfile`:

```docker
ARG  CODE_VERSION=latest
FROM base:${CODE_VERSION}
CMD  /code/run-app
```

When building a Docker image from the commandline, you can set `ARG` values using `--build-arg`:

```bash
docker build --build-arg CODE_VERSION=4.7.2
```

## References
- [Docker `ARG` official docs](https://docs.docker.com/engine/reference/builder/#arg)
