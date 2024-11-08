### Deep Dive into Static Pods in Kubernetes

---

#### Table of Contents
1. [Introduction to Static Pods](#introduction-to-static-pods)
2. [The Role of Kubelet in Managing Static Pods](#the-role-of-kubelet-in-managing-static-pods)
3. [How Static Pods are Identified](#how-static-pods-are-identified)
4. [Understanding the Path for Static Pods](#understanding-the-path-for-static-pods)
5. [Locating the Static Pod Path Using Kubelet Configuration](#locating-the-static-pod-path-using-kubelet-configuration)
6. [Creating a Static Pod Using kubectl](#creating-a-static-pod-using-kubectl)
7. [Identifying and Deleting a Static Pod on a Node](#identifying-and-deleting-a-static-pod-on-a-node)
8. [Managing Static Pods via Configuration Files](#managing-static-pods-via-configuration-files)
9. [Using Runtime Commands for Static Pods](#using-runtime-commands-for-static-pods)
10. [Examples of Static Pods](#examples-of-static-pods)
11. [Using `-o yaml` to Identify Static Pods](#using--o-yaml-to-identify-static-pods)
12. [Conclusion](#conclusion)

---

### Introduction to Static Pods

**Static Pods** are a special type of pod in Kubernetes managed directly by the **kubelet** on individual nodes rather than being scheduled by the Kubernetes API server. Static Pods are commonly used to run critical system components such as `etcd`, `kube-apiserver`, `kube-controller-manager`, and `kube-scheduler`, ensuring these pods are bound to specific nodes and continue running even if the API server is unavailable.

---

### The Role of Kubelet in Managing Static Pods

The **kubelet** is responsible for managing the lifecycle of Static Pods. It monitors a specific directory (usually `/etc/kubernetes/manifests/`) on each node’s filesystem for any pod manifest files. When a new manifest file is added to this directory, the kubelet automatically creates the pod. If the file is modified or deleted, the kubelet updates or stops the pod accordingly.

---

### How Static Pods are Identified

Static Pods can be identified by two main characteristics:

1. **Node-Specific Naming**: Static Pods are named based on the node they are running on. For instance, if a node is named `controlplane`, a Static Pod defined by a file called `kube-apiserver.yaml` will appear as `kube-apiserver-controlplane` in the Kubernetes API.

2. **Mirror Pods**: Static Pods are visible in the Kubernetes API as **mirror pods**, which are read-only representations created by the kubelet. These mirror pods allow administrators to monitor static pods from the Kubernetes API, but their configuration cannot be changed through the API itself.

---

### Understanding the Path for Static Pods

Static Pods are defined by manifest files stored in directories such as `/etc/kubernetes/manifests`. However, the exact location of these files can vary depending on the node's configuration. Each kubelet is configured to look in a specified path for Static Pod manifests, which can be customized per node.

This is important when troubleshooting or managing Static Pods, as you may need to connect to the specific node (such as `node01`) to confirm the correct path if it's not the standard `/etc/kubernetes/manifests`. This helps to ensure you are looking in the right directory when creating, modifying, or deleting static pod manifest files.

---

### Locating the Static Pod Path Using Kubelet Configuration

Since the path where static pods are stored may vary, you can check the kubelet configuration on a given node to determine where it’s looking for static pod files. Here’s how to identify this path:

1. **Connect to the Node**: If you’re managing a pod on a specific node, such as `node01`, SSH into that node:

   ```bash
   root@controlplane:~# ssh node01
   ```

2. **Identify the Kubelet Configuration Path**: Use the following command to view the kubelet’s configuration path:

   ```bash
   root@node01:~# ps -ef | grep -i kubelet
   ```

   Example output:

   ```plaintext
   root        4147       1  0 14:05 ?        00:00:00 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock --pod-infra-container-image=registry.k8s.io/pause:3.9
   ```

   Here, the `--config` flag shows that the kubelet’s configuration file is located at `/var/lib/kubelet/config.yaml`.

3. **Check the Static Pod Path in the Configuration File**: Open or search the kubelet configuration file to find the `staticPodPath` setting, which defines where the kubelet looks for static pod files:

   ```bash
   root@node01:~# grep -i staticpod /var/lib/kubelet/config.yaml
   ```

   Example output:

   ```plaintext
   staticPodPath: /etc/just-to-mess-with-you
   ```

   This confirms that the kubelet on `node01` monitors `/etc/just-to-mess-with-you` for static pod manifests instead of the default `/etc/kubernetes/manifests`.

By using this approach, you can ensure you’re managing static pods in the correct directory, especially if your cluster setup deviates from the standard configuration.

---

### Creating a Static Pod Using kubectl

To create a static pod manifest, you can use `kubectl run` with the `--dry-run` option and redirect the output to the appropriate directory:

```bash
kubectl run static-busybox --image=busybox --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml
```

Once the file is placed in the manifest directory, the kubelet will automatically create the static pod.

---

### Identifying and Deleting a Static Pod on a Node

To identify a static pod on a node, you can query for the node it’s running on:

```bash
kubectl get pods --all-namespaces -o wide | grep static-greenbox
```

Example output:

```plaintext
default       static-greenbox-node01   1/1   Running   0   19s   10.244.1.2   node01   <none>   <none>
```

The static pod will have the node’s name in its name (e.g., `static-greenbox-node01`). To delete this static pod, remove its manifest file from the specified path. 

1. SSH into the node where the pod is running:

   ```bash
   root@controlplane:~# ssh node01
   ```

2. Navigate to the static pod path:

   ```bash
   root@node01:/etc/just-to-mess-with-you# ls
   ```

3. Delete the manifest file:

   ```bash
   root@node01:/etc/just-to-mess-with-you# rm -rf greenbox.yaml
   ```

4. Exit from `node01`:

   ```bash
   root@node01:~# exit
   ```

5. Verify the deletion on the control plane:

   ```bash
   root@controlplane:~# kubectl get pods --all-namespaces -o wide | grep static-greenbox
   ```

If the deletion was successful, there will be no output.

---

### Managing Static Pods via Configuration Files

Static Pods are entirely managed by their configuration files, usually stored in the path specified by `staticPodPath` in the kubelet’s configuration (often `/etc/kubernetes/manifests`).

---

### Using Runtime Commands for Static Pods

If `kubectl` is not sufficient, you can use runtime-specific commands, such as `crictl` or `nerdctl`, to interact with static pods directly at the container runtime level.

---

### Examples of Static Pods

Below is an example manifest for a static pod:

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

To check if a pod is static, use the following command:

```bash
kubectl get pod <pod-name> -o yaml
```

The output will include the `ownerReferences` field:

- **Static Pods**: `ownerReferences` will either be missing or show `kind: Node`, indicating it’s managed by the kubelet.
  
  ```yaml
  ownerReferences:
  - apiVersion: v1
    kind: Node
    name: node01
  ```

- **Dynamically Managed Pods**: `ownerReferences` will show a controller such as `ReplicaSet` or `Deployment`.

  ```yaml
  ownerReferences:
  - apiVersion: apps/v1
    kind: ReplicaSet
    name: dynamic-app-frontend-5b6f77f9b4
  ```

---

### Conclusion

Static Pods provide a reliable way to manage critical node-level services directly through the kubelet. Understanding their configuration, identifying the correct paths, and knowing how to locate and manage static pod files allows Kubernetes administrators to ensure the

 reliability of essential services in a cluster.
