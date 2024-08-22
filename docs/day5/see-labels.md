### Command to See Labels 

Note: Importan for certification

```bash
kubectl get pods --show-labels
```

This will output a list of Pods with their corresponding labels in the current namespace. If you want to see labels for Pods in a different namespace, you can specify the namespace using the `-n` flag:

```bash
kubectl get pods -n <namespace> --show-labels
```

Replace `<namespace>` with the name of the namespace you want to query.
