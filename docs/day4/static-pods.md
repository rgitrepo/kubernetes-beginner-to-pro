Here’s the updated tutorial focused solely on **Static Pods in Kubernetes**, with more detailed explanations and examples, while removing the focus on DaemonSets and Deployments.

---

### Tutorial: Deep Dive into Static Pods in Kubernetes

#### Table of Contents
1. [Introduction to Static Pods](#introduction-to-static-pods)
2. [The Role of Kubelet in Managing Static Pods](#the-role-of-kubelet-in-managing-static-pods)
3. [How Static Pods are Identified](#how-static-pods-are-identified)
4. [Understanding the Path for Static Pods](#understanding-the-path-for-static-pods)
5. [Managing Static Pods via Configuration Files](#managing-static-pods-via-configuration-files)
6. [Modifying the Static Pod Path and Kubelet Configuration](#modifying-the-static-pod-path-and-kubelet-configuration)
7. [Using Runtime Commands for Static Pods](#using-runtime-commands-for-static-pods)
8. [Examples of Static Pods](#examples-of-static-pods)
9. [Conclusion](#conclusion)

---

### Introduction to Static Pods

**Static Pods** are a special type of pod in Kubernetes that are directly managed by the **kubelet** on individual nodes, rather than being scheduled by the Kubernetes API server. Static Pods are typically used for critical system components such as `etcd`, `kube-apiserver`, `kube-controller-manager`, and `kube-scheduler`, especially in self-hosted or bootstrapped clusters.

Unlike regular pods, which are orchestrated by the Kubernetes API server, Static Pods are created, monitored, and managed entirely by the kubelet running on the node.

---

### The Role of Kubelet in Managing Static Pods

The **kubelet** is responsible for managing the lifecycle of Static Pods. The kubelet continuously watches a specific directory (e.g., `/etc/kubernetes/manifests/`) on the node’s filesystem for any pod manifests. When a new manifest file is added, the kubelet will create the pod, and if the file is modified or deleted, the kubelet will update or stop the pod accordingly.

**Key Points**:
- Static Pods are not managed by the Kubernetes API server.
- The kubelet ensures that Static Pods are always running and will restart them if they fail or are removed.

---

### How Static Pods are Identified

Static Pods are identified and managed differently from regular Kubernetes-managed pods. Here are some key identifiers:

1. **Node-Specific Naming**:
   Static Pods are named based on the node where they are running. If a node is named `controlplane`, a Static Pod defined by a file called `kube-apiserver.yaml` will appear in the Kubernetes API as `kube-apiserver-controlplane`.

   **Example**:
   - Node: `controlplane`
   - Pod Manifest: `kube-apiserver.yaml`
   - The pod will be displayed as `kube-apiserver-controlplane` when using:
     ```bash
     kubectl get pods -n kube-system
     ```
     **Output Example**:
     ```
     NAME                         READY   STATUS    RESTARTS   AGE
     kube-apiserver-controlplane  1/1     Running   0          5m
     ```

2. **Mirror Pods**:
   Static Pods are not created by the Kubernetes API server, but they are visible in the API as **mirror pods**. These mirror pods provide visibility and tracking through the API but are read-only, meaning they cannot be modified or deleted using `kubectl`.

3. **Pod Lifecycle Management**:
   Since the kubelet manages Static Pods, any changes to the pod’s lifecycle (creation, modification, or deletion) are handled directly by the kubelet based on the manifest files in the specified directory.

---

### Understanding the Path for Static Pods

Static Pods are defined using pod manifest files located in a specific directory on the node, typically `/etc/kubernetes/manifests`. The kubelet continuously monitors this directory, and any YAML files placed here will automatically be created as Static Pods.

**Steps**:
1. **Locate the Static Pod Path**:
   By default, the manifest files for Static Pods are stored in `/etc/kubernetes/manifests`.
   ```bash
   cd /etc/kubernetes/manifests
   ls
   ```
   This command will list all the static pod manifest files.

2. **View a Static Pod Manifest**:
   You can inspect the contents of a static pod manifest file to understand its configuration:
   ```bash
   cat kube-apiserver.yaml
   ```

3. **Pod Creation**:
   When you add a valid pod manifest file to this directory, the kubelet will automatically create the corresponding pod.

---

### Managing Static Pods via Configuration Files

Static Pods are entirely managed via configuration files stored in the specified manifest directory on each node. These files are in YAML format, similar to regular Kubernetes pod definitions, and describe the static pod’s configuration.

**Steps to Manage Static Pods**:

1. **Create or Modify a Static Pod**:
   Static Pods are managed by editing or creating YAML manifest files. For example, if you want to modify a Static Pod’s configuration, you would:
   - Open the manifest file:
     ```bash
     nano /etc/kubernetes/manifests/kube-apiserver.yaml
     ```
   - Make your changes (e.g., update the container image or change the ports).
   - Save the file. The kubelet will automatically detect the changes and restart the pod with the new configuration.

2. **Delete a Static Pod**:
   To delete a static pod, simply remove its manifest file from the `/etc/kubernetes/manifests` directory:
   ```bash
   sudo rm /etc/kubernetes/manifests/kube-apiserver.yaml
   ```
   Once the file is removed, the kubelet will stop the pod.

3. **No `kubectl` Management**:
   Since static pods are not managed by the Kubernetes API, `kubectl` commands like `kubectl delete` or `kubectl apply` will not work for static pods. The only way to manage them is via the configuration files on the node.

---

### Modifying the Static Pod Path and Kubelet Configuration

By default, the kubelet monitors the `/etc/kubernetes/manifests` directory for static pod manifests. However, you can modify the kubelet’s configuration to point to a different directory if needed. The kubelet’s configuration is typically found in `/var/lib/kubelet/config.yaml`.

**Steps to Change the Static Pod Path**:

1. **Edit the Kubelet Configuration**:
   Open the kubelet configuration file:
   ```bash
   nano /var/lib/kubelet/config.yaml
   ```

2. **Modify the Static Pod Path**:
   Look for the `staticPodPath` entry in the configuration file. You can change it to a custom directory:
   ```yaml
   staticPodPath: "/custom/static-pods"
   ```

3. **Restart the Kubelet**:
   After making the change, restart the kubelet service to apply the changes:
   ```bash
   systemctl restart kubelet
   ```

4. **Verify the Changes**:
   Add a static pod manifest to the new directory and verify the pod is created:
   ```bash
   kubectl get pods -n kube-system
   ```

---

### Using Runtime Commands for Static Pods

Since Static Pods are managed by the kubelet, some standard `kubectl` commands might not work to inspect or troubleshoot these pods. Instead, runtime-specific commands may be required depending on the container runtime being used.

1. **For CRI-O**:
   Use `crictl` to interact with containers running in CRI-O:
   ```bash
   crictl ps
   ```

2. **For containerd**:
   Use `nerdctl` to inspect containers managed by containerd:
   ```bash
   nerdctl ps
   ```

These runtime commands allow you to inspect, debug, and manage containers that are part of Static Pods when standard `kubectl` commands are not sufficient.

---

### Examples of Static Pods

Here is an example of a static pod manifest for the `etcd` component:

**etcd Static Pod Example**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: etcd
  namespace: kube-system
spec:
  containers:
  - name: etcd
    image: k8s.gcr.io/etcd:3.4.13-0
    command:
    - etcd
    - --advertise-client-urls=http://localhost:2379
    - --listen-client-urls=http://localhost:2379
    ports:
    - containerPort: 2379
      protocol: TCP
```

**Steps to Deploy**:
1. Save this manifest file as `etcd.yaml`.
2. Place the file in the `/etc/kubernetes/manifests/` directory:
   ```bash
   sudo mv etcd.yaml /etc/kubernetes/manifests/
   ```
3. The kubelet will automatically create the static pod and run `etcd`.

4. Verify the pod is running:
   ```bash
   kubectl get pods -n kube-system
   ```

---

### Conclusion

Static Pods are a powerful feature in Kubernetes, providing a way to manage critical node-level services independently of the Kubernetes API server. By relying on the kubelet for lifecycle management, Static Pods ensure high availability and resiliency for system-critical components like `etcd` and `kube-apiserver`. With the ability to modify configurations through manifest files and the need for runtime-specific tools to debug, understanding how to work with Static Pods is essential for Kubernetes administrators managing highly available clusters

