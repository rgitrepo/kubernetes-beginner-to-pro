### Tutorial: Deep Dive into Static Pods in Kubernetes

---

#### Table of Contents
1. [Introduction to Static Pods](#introduction-to-static-pods)
2. [The Role of Kubelet in Managing Static Pods](#the-role-of-kubelet-in-managing-static-pods)
3. [How Static Pods are Identified](#how-static-pods-are-identified)
4. [Understanding the Path for Static Pods](#understanding-the-path-for-static-pods)
5. [Creating a Static Pod Using kubectl](#creating-a-static-pod-using-kubectl)
6. [Identifying and Deleting a Static Pod on a Node](#identifying-and-deleting-a-static-pod-on-a-node)
7. [Managing Static Pods via Configuration Files](#managing-static-pods-via-configuration-files)
8. [Using Runtime Commands for Static Pods](#using-runtime-commands-for-static-pods)
9. [Examples of Static Pods](#examples-of-static-pods)
10. [Using `-o yaml` to Identify Static Pods](#using--o-yaml-to-identify-static-pods)
11. [Conclusion](#conclusion)

---

### Introduction to Static Pods

**Static Pods** are a special type of pod in Kubernetes that are directly managed by the **kubelet** on individual nodes, rather than being scheduled by the Kubernetes API server. Static Pods are typically used for critical system components such as `etcd`, `kube-apiserver`, `kube-controller-manager`, and `kube-scheduler`.

---

### The Role of Kubelet in Managing Static Pods

The **kubelet** is responsible for managing the lifecycle of Static Pods. It continuously watches a specific directory (e.g., `/etc/kubernetes/manifests/`) on the node’s filesystem for any pod manifests. When a new manifest file is added, the kubelet will create the pod, and if the file is modified or deleted, the kubelet will update or stop the pod accordingly.

---

### How Static Pods are Identified

1. **Node-Specific Naming**: Static Pods are named based on the node where they are running. For example, if a node is named `controlplane`, a Static Pod defined by a file called `kube-apiserver.yaml` will appear as `kube-apiserver-controlplane`.

2. **Mirror Pods**: Static Pods are visible in the Kubernetes API as **mirror pods** but are read-only.

---

### Understanding the Path for Static Pods

Static Pods are defined using manifest files located in directories like `/etc/kubernetes/manifests`. The kubelet monitors this directory and automatically manages the pods based on the files found here.

---

### Creating a Static Pod Using kubectl

To create a static pod manifest, you can use `kubectl run` and redirect the output to the appropriate directory:

```bash
kubectl run static-busybox --image=busybox --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml
```

Once the file is placed in the manifest directory, the kubelet will automatically create the static pod.

---

### Identifying and Deleting a Static Pod on a Node

You can identify a static pod by querying the node it runs on:

```bash
kubectl get pods --all-namespaces -o wide | grep static-greenbox
```

Example output:
```bash
default       static-greenbox-node01   1/1   Running   0   19s   10.244.1.2   node01   <none>   <none>
default       dynamic-app-frontend     2/2   Running   0   10m   10.244.1.10  node01   <none>   <none>
default       dynamic-app-backend      1/1   Running   0   8m    10.244.1.12  node02   <none>   <none>
```
The static pod ends with the node name, whereas dynamically managed pods (`frontend`, `backend`) don't.

To delete a static pod, remove its manifest from the node's manifest directory.

---

### Managing Static Pods via Configuration Files

Static Pods are fully managed via their configuration files, usually stored in `/etc/kubernetes/manifests/`.

---

### Using Runtime Commands for Static Pods

When `kubectl` isn't enough, you can use runtime-specific commands like `crictl` or `nerdctl` to interact with containers.

---

### Examples of Static Pods

Here’s an example manifest of a static pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: static-busybox
  namespace: kube-system
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sleep", "1000"]
```

---

### Using `-o yaml` to Identify Static Pods

To check if a pod is static, use the `kubectl get pod <pod-name> -o yaml` command. This will display the full configuration of the pod, including the `ownerReferences` field, which is key for determining how the pod is managed:

- **Static Pods**: The `ownerReferences` field will either be missing or have the `kind: Node`. For example:

  ```yaml
  ownerReferences:
  - apiVersion: v1
    kind: Node
    name: node01
  ```

- **Dynamically Managed Pods**: The `ownerReferences` field will indicate a controller such as `ReplicaSet`, `Deployment`, or `StatefulSet`. For example:

  ```yaml
  ownerReferences:
  - apiVersion: apps/v1
    kind: ReplicaSet
    name: dynamic-app-frontend-5b6f77f9b4
  ```

This key difference helps identify whether a pod is static or dynamically managed by Kubernetes.

---

### Conclusion

Static Pods offer a way to manage critical node-level services. Understanding their relationship with the kubelet, and how to identify and manage them, provides administrators with the tools needed to ensure the reliability of critical system services in Kubernetes clusters.
