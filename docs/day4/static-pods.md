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
10. [Conclusion](#conclusion)

---

### Introduction to Static Pods

**Static Pods** are a special type of pod in Kubernetes that are directly managed by the **kubelet** on individual nodes, rather than being scheduled by the Kubernetes API server. Static Pods are typically used for critical system components such as `etcd`, `kube-apiserver`, `kube-controller-manager`, and `kube-scheduler`.

Unlike regular pods, which are orchestrated by the Kubernetes API server, Static Pods are created, monitored, and managed entirely by the kubelet running on the node.

---

### The Role of Kubelet in Managing Static Pods

The **kubelet** is responsible for managing the lifecycle of Static Pods. It continuously watches a specific directory (e.g., `/etc/kubernetes/manifests/`) on the node’s filesystem for any pod manifests. When a new manifest file is added, the kubelet will create the pod, and if the file is modified or deleted, the kubelet will update or stop the pod accordingly.

**Key Points**:
- Static Pods are not managed by the Kubernetes API server.
- The kubelet ensures that Static Pods are always running and will restart them if they fail or are removed.

---

### How Static Pods are Identified

Static Pods are identified and managed differently from regular Kubernetes-managed pods. Here are some key identifiers:

1. **Node-Specific Naming**:
   Static Pods are named based on the node where they are running. If a node is named `controlplane`, a Static Pod defined by a file called `kube-apiserver.yaml` will appear in the Kubernetes API as `kube-apiserver-controlplane`.

2. **Mirror Pods**:
   Static Pods are visible in the Kubernetes API as **mirror pods**. These mirror pods provide visibility and tracking through the API but are read-only, meaning they cannot be modified or deleted using `kubectl`.

3. **Pod Lifecycle Management**:
   Since the kubelet manages Static Pods, any changes to the pod’s lifecycle (creation, modification, or deletion) are handled directly by the kubelet based on the manifest files in the specified directory.

---

### Understanding the Path for Static Pods

Static Pods are defined using pod manifest files located in a specific directory on the node, typically `/etc/kubernetes/manifests`. The kubelet continuously monitors this directory, and any YAML files placed here will automatically be created as Static Pods.

---

### Creating a Static Pod Using kubectl

You can generate a static pod manifest using the `kubectl` command with the `--dry-run` option. Here’s an example that creates a static pod running **busybox** that sleeps for 1000 seconds.

**Command**:
```bash
kubectl run --restart=Never --image=busybox static-busybox --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml
```

**Explanation**:
1. **`--restart=Never`**: This ensures that the pod is created as a standalone pod rather than as part of a controller like a Deployment or ReplicaSet.
2. **`--image=busybox`**: Specifies the image for the container (in this case, `busybox`).
3. **`--dry-run=client`**: Ensures that no resources are created immediately, and instead the output is formatted as a YAML file.
4. **`-o yaml`**: Outputs the manifest in YAML format.
5. **`--command -- sleep 1000`**: Specifies the command the container will run (in this case, the container will sleep for 1000 seconds).
6. **Redirect to `/etc/kubernetes/manifests/`**: The generated YAML file is saved directly into the `/etc/kubernetes/manifests/` directory. Since this directory is monitored by the kubelet, the pod will automatically be created once the file is saved.

**Result**:
- Once the YAML file is placed in `/etc/kubernetes/manifests/`, the kubelet will detect it and immediately create the **static-busybox** pod.

---

### Identifying and Deleting a Static Pod on a Node

In Kubernetes, you can identify which node a static pod is running on, and you can SSH into that node to manage the pod's configuration file directly. Here's an example process for identifying and deleting a static pod.

#### Step 1: Identify the Node Running the Static Pod

First, we need to find out which node is running the static pod. For this example, let's assume the pod is called `static-greenbox`.

**Command**:
```bash
kubectl get pods --all-namespaces -o wide | grep static-greenbox
```

**Example Output**:
```bash
default       static-greenbox-node01   1/1   Running   0   19s   10.244.1.2   node01   <none>   <none>
```

From the output, we can see that the pod `static-greenbox-node01` is running on the node called `node01`.

#### Step 2: SSH into the Node

Once we know the pod is running on `node01`, we SSH into the node to investigate its configuration.

**Command**:
```bash
ssh node01
```

#### Step 3: Identify the Static Pod Path in the Kubelet Configuration

While the default path for static pod manifests is usually `/etc/kubernetes/manifests`, this may differ based on how the node is configured. To find the path being used, we can inspect the kubelet's running configuration.

**Command to find the kubelet process**:
```bash
ps -ef | grep /usr/bin/kubelet
```

Example Output:
```bash
root        4147       1  0 14:05 ?        00:00:00 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock --pod-infra-container-image=registry.k8s.io/pause:3.9
```

From this output, we can see that the kubelet is using the configuration file located at `/var/lib/kubelet/config.yaml`.

Next, we can check this configuration file for the static pod path.

**Command to check static pod path**:
```bash
grep -i staticpod /var/lib/kubelet/config.yaml
```

Example Output:
```bash
staticPodPath: /etc/just-to-mess-with-you
```

In this case, we can see that the static pod path is set to `/etc/just-to-mess-with-you`.

#### Step 4: Navigate to the Static Pod Directory

Now that we know the directory where the static pod manifests are stored, we can navigate to it.

**Command**:
```bash
cd /etc/just-to-mess-with-you
ls
```

Example Output:
```bash
greenbox.yaml
```

We can see the static pod manifest for `greenbox.yaml`.

#### Step 5: Delete the Static Pod Manifest

To delete the pod, simply remove its manifest file from the directory.

**Command**:
```bash
rm -rf greenbox.yaml
```

#### Step 6: Verify Pod Deletion

Once the manifest file is removed, the kubelet will automatically stop the pod. Exit the node and verify that the pod has been deleted.

**Command**:
```bash
kubectl get pods --all-namespaces -o wide | grep static-greenbox
```

Example Output:
```bash
# No output means the pod has been successfully deleted.
```

---

### Managing Static Pods via Configuration Files

Static Pods are entirely managed via configuration files stored in the `/etc/kubernetes/manifests/` (or another path defined by the kubelet configuration). These files are in YAML format and describe the static pod’s configuration.

---

### Using Runtime Commands for Static Pods

Since Static Pods are managed by the kubelet, standard `kubectl` commands might not be sufficient to inspect or troubleshoot these pods. Instead, runtime-specific commands may be required depending on the container runtime being used.

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

Here is an example of a static pod manifest for the **busybox** pod created earlier:

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

This YAML file describes a static pod that runs a **busybox** container and sleeps for 1000 seconds.

---

### Conclusion

Static Pods provide a powerful way to manage critical node-level services independently of the Kubernetes API server. By leveraging the kubelet for lifecycle management and using the `/etc/kubernetes/manifests/` directory (or any configured path) to define pods, Static Pods ensure high availability for system-critical components.

Understanding how to create, manage, and delete Static Pods—along with how to identify nodes where they run—enables Kubernetes administrators to build resilient, self-hosting clusters with ease.

