### Tutorial: Overriding CMD and ENTRYPOINT in Kubernetes Pod Definition YAML

In Kubernetes, the `command` and `args` fields in a pod's definition can override the `ENTRYPOINT` and `CMD` instructions in the Dockerfile, respectively. This gives you flexibility in how a container runs by changing its behavior without modifying the container image itself.

#### Dockerfile Example

Let's start with a Dockerfile that has both `ENTRYPOINT` and `CMD`.

```Dockerfile
# Dockerfile
FROM busybox

# Set ENTRYPOINT to "sleep"
ENTRYPOINT ["sleep"]

# Set CMD to "5"
CMD ["5"]
```

- `ENTRYPOINT`: This specifies the executable to run, in this case, `"sleep"`.
- `CMD`: This provides default arguments to the executable, here it is `"5"`, meaning the container will sleep for 5 seconds by default.

#### Pod Definition YAML

In Kubernetes, we can override both the `ENTRYPOINT` and `CMD` using the `command` and `args` fields in the pod definition YAML.

```yaml
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: override-example
spec:
  containers:
  - name: sleep-container
    image: busybox
    command: ["sleep2.0"]  # This overrides ENTRYPOINT ["sleep"] in the Dockerfile
    args: ["10"]           # This overrides CMD ["5"] in the Dockerfile
  restartPolicy: Never
```

- **command**: This overrides the `ENTRYPOINT` from the Dockerfile. In this case, `"sleep2.0"` replaces the default `"sleep"`.
- **args**: This overrides the `CMD` from the Dockerfile. `"10"` replaces the default argument `"5"`, meaning the container will now sleep for 10 seconds instead of 5.

#### How It Works

- If the Dockerfile defines `ENTRYPOINT` as `"sleep"` and `CMD` as `"5"`, the container would normally run as:
  ```bash
  sleep 5
  ```

- However, with the pod YAML overriding both `ENTRYPOINT` and `CMD`, the container will instead run:
  ```bash
  sleep2.0 10
  ```

### Testing the Pod

To deploy the pod:

```bash
kubectl apply -f pod-definition.yaml
```

Check the pod's logs to verify that the `command` and `args` were applied correctly:

```bash
kubectl logs override-example
```

In the logs, you should see that the pod ran with the overridden values (`sleep2.0 10`).

### Summary

- `command` in the YAML overrides the Dockerfile's `ENTRYPOINT`.
- `args` in the YAML overrides the Dockerfile's `CMD`.

This is a useful mechanism when you want to adjust the behavior of your container at runtime without modifying the Docker image itself.
