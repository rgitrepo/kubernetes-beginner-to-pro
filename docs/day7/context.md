

## Kubernetes Contexts

### Table of Contents

- [Introduction to Kubernetes Contexts](#introduction-to-kubernetes-contexts)
- [Viewing and Setting Contexts](#viewing-and-setting-contexts)
- [Creating and Changing Contexts](#creating-and-changing-contexts)
- [Using Different Contexts to Query Resources](#using-different-contexts-to-query-resources)
- [Understanding MySQL Connection Strings in Kubernetes](#understanding-mysql-connection-strings-in-kubernetes)
- [Best Practices for Managing Contexts](#best-practices-for-managing-contexts)
- [Summary](#summary)

---

### Introduction to Kubernetes Contexts

Kubernetes contexts define a set of access parameters for connecting to a Kubernetes cluster, including the cluster, user, and namespace. This allows you to manage multiple clusters and environments easily.

### Viewing and Setting Contexts

To view the current context, use:

```bash
kubectl config current-context
```

To set the current context, run:

```bash
kubectl config use-context my-cluster-context
```

### Creating and Changing Contexts

To create a new context, use:

```bash
kubectl config set-context dev-context --cluster=my-cluster --user=my-user --namespace=development
```

To update the current context's default namespace, use:

```bash
kubectl config set-context $(kubectl config current-context) --namespace=development
```

- **`$(kubectl config current-context)`** retrieves the name of the current context, updating its default namespace to `development`.

### Using Different Contexts to Query Resources

After setting up contexts, switch between them using:

```bash
kubectl config use-context prod-context
```

For example, after switching to a context named `prod-context`, running:

```bash
kubectl get pods
```

will list the pods in the default namespace of that context.

### Understanding MySQL Connection Strings in Kubernetes

When connecting to a MySQL database in Kubernetes, you might use a connection string like this:

```python
mysql.connect("db-service.dev.svc.cluster.local")
```

#### Breakdown of the DNS Components:

- **`cluster.local`**: This is the default domain for services in a Kubernetes cluster, indicating that the service is within the cluster.
- **`svc`**: Stands for "service," denoting that this is a service endpoint.
- **`dev`**: This specifies the namespace where the service resides. In this case, it's the `dev` namespace.
- **`db-service`**: This is the name of the service that you're trying to connect to.

#### Comparison with Short Name Usage

If you are already in the same namespace (e.g., `dev`), you can simplify the connection string:

```python
mysql.connect("db-service")
```

Using the short name is valid because Kubernetes resolves service names to their respective IP addresses within the same namespace automatically.

### Best Practices for Managing Contexts

1. **Descriptive Names**: Use clear names for contexts, clusters, and users.
2. **Regular Review**: Check existing contexts with `kubectl config get-contexts` to manage them effectively.
3. **Namespace Awareness**: Be mindful of the default namespace in your current context.
4. **Direct Namespace Queries**: Use the `--namespace` flag in commands if you need to override the default.

### Summary

Kubernetes contexts simplify management when working with multiple clusters and environments. By understanding how to set and switch contexts, you can streamline your workflow and ensure proper resource management.

