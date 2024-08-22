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
