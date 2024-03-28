# Define a Command and Arguments for a Container

This section shows how to define commands and arguments when you run a container in a Pod.

## Define a command and arguments when you create a Pod

When you create a Pod, you can define a command and arguments for the containers that run in the Pod.

The command and arguments that you define cannot be changed after the Pod is created.

**The command and arguments that you define in the configuration file override the default command and arguments provided by the container image**. If you define args, but do not define a command, the default command is used with your new arguments.

- To define a command, include the `command` field in the configuration file. 
- To define arguments for the command, include the `args` field in the configuration file.

!!! note
    The `command` field corresponds to entrypoint in some container runtimes.

For example, let's create a Pod with one container that defines a command and two arguments:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: command-demo
spec:
  containers:
  - name: command-demo-container
    image: debian
    command: ["printenv"]
    args: ["HOSTNAME", "KUBERNETES_PORT"]
  restartPolicy: OnFailure
```

Once the Pod is created (and is up and running), view the logs from the Pod:

```bash
kubectl logs command-demo
```

The output shows the values of the `HOSTNAME` and `KUBERNETES_PORT` environment variables:

```bash
command-demo
tcp://10.254.0.1:443
```


## Use environment variables to define arguments

In the preceding example, you defined the arguments directly by providing strings. As an alternative to providing strings directly, you can define arguments by using environment variables:


```yaml
env:
- name: MESSAGE
  value: "hello world"
command: ["/bin/echo"]
args: ["$(MESSAGE)"]
```

!!! note
    The environment variable appears in parentheses, `"$(VAR)"`. This is required for the variable to be expanded in the `command` or `args` field.

## Run a command in a shell

In some cases, you need your command to run in a shell. For example, your command might consist of several commands piped together, or it might be a shell script. To run your command in a shell, wrap it like this:

```yaml
command: ["/bin/sh"]
args: ["-c", "while true; do echo hello; sleep 10;done"]
```
