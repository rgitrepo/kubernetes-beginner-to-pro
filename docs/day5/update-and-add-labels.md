## Update and Add Labels

The command you've mentioned is close, but it has a few issues. Here's the correct command to update an existing label and add a new label to a Kubernetes Pod:

### To Update an Existing Label and Add a New Label:
The correct command structure is as follows:

```bash
kubectl label pod <pod-name> <existing-label-key>=<updated-value> <new-label-key>=<new-label-value>
```

- `<pod-name>`: The name of the Pod you want to label.
- `<existing-label-key>`: The key of the label that already exists on the Pod and needs to be updated.
- `<updated-value>`: The new value you want to assign to the existing label.
- `<new-label-key>`: The key of the new label you want to add to the Pod.
- `<new-label-value>`: The value of the new label you want to add.

### Example:
Suppose you have a Pod named `mypod`, and it has an existing label `env=production` that you want to update to `env=staging`. You also want to add a new label `tier=backend`.

The command would be:

```bash
kubectl label pod mypod env=staging tier=backend
```

### Explanation:
- This command updates the `env` label to `staging`.
- It also adds a new label `tier=backend` to the `mypod`.

### Important Notes:
- If you run the command with an existing label key and a new value, it will update that label with the new value.
- If you specify a new label key that doesn't exist, it will add that new label to the Pod.
- Kubernetes will overwrite the existing label value if the key already exists with a different value.

You can verify the labels on your Pod by running:

```bash
kubectl get pods <pod-name> --show-labels
```

This command will display all the labels associated with the specified Pod.


### Synopsis
Update the labels on a resource.

A label key and value must begin with a letter or number, and may contain letters, numbers, hyphens, dots, and underscores, up to 63 characters each.
Optionally, the key can begin with a DNS subdomain prefix and a single '/', like example.com/my-app.
If --overwrite is true, then existing labels can be overwritten, otherwise attempting to overwrite a label will result in an error.
If --resource-version is specified, then updates will use this resource version, otherwise the existing resource-version will be used.
kubectl label [--overwrite] (-f FILENAME | TYPE NAME) KEY_1=VAL_1 ... KEY_N=VAL_N [--resource-versio

```bash
  kubectl label [--overwrite] (-f FILENAME | TYPE NAME) KEY_1=VAL_1 ... KEY_N=VAL_N [--resource-version=version]
```

### Examples

```bash
  # Update pod 'old-pod' with the label 'unhealthy' and the value 'true'
  kubectl label pods old-pod foo unhealthy=true
  
  # Update pod 'old-pod' with the label 'status' and the value 'unhealthy', overwriting any existing value
  kubectl label --overwrite pods old-pod status=unhealthy
  
  # Update all pods in the namespace
  kubectl label pods --all status=unhealthy
  
  # Update a pod identified by the type and name in "pod.json"
  kubectl label <strong>-f <\strong> pod.json status=unhealthy
  
  # Update pod 'foo' only if the resource is unchanged from version 1
  kubectl label pods foo status=unhealthy --resource-version=1
  
  # Update pod 'foo' by removing a label named 'bar' if it exists
  # Does not require the --overwrite flag
  kubectl label pods foo bar-

  ```
Options
