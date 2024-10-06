### Kubernetes Upgrade Tutorial: Control Plane and Worker Node to v1.30.0

This tutorial will guide you through upgrading both the **control plane node** (`controlplane`) and a **worker node** (`node01`) to Kubernetes v1.30.0. The process differs slightly between control plane and worker nodes, and this guide will point out those differences at the relevant steps. Additional details about the `--ignore-daemonsets` and `--force` options are provided for clarity.

---

#### **Table of Contents**

### Control Plane Node (`controlplane`)
1. [Upgrade `kubeadm` on the Control Plane Node](#1-upgrade-kubeadm-on-the-control-plane-node-controlplane)
2. [Drain the Control Plane Node](#2-drain-the-control-plane-node-controlplane)
3. [Upgrade Control Plane Components](#3-upgrade-control-plane-components-controlplane)
4. [Upgrade `kubelet` on the Control Plane Node](#4-upgrade-kubelet-on-the-control-plane-node-controlplane)

### Worker Node (`node01`)
5. [Upgrade `kubeadm` on the Worker Node](#5-upgrade-kubeadm-on-the-worker-node-node01)
6. [Drain the Worker Node](#6-drain-the-worker-node-node01)
7. [Upgrade Kubelet on the Worker Node](#7-upgrade-kubelet-on-the-worker-node-node01)
8. [Mark the Worker Node as Schedulable](#8-mark-the-worker-node-as-schedulable)

### Final Steps and Verification
9. [Verification for Both Nodes](#9-verification-for-both-nodes)
10. [Additional Notes on `--ignore-daemonsets` and `--force`](#10-additional-notes-on-ignore-daemonsets-and-force)

---

### Control Plane Node (`controlplane`)

### 1. Upgrade `kubeadm` on the Control Plane Node (`controlplane`)
The first step is to upgrade `kubeadm` on the control plane node. Ensure the Kubernetes APT repository is updated to point to the new version.

1. Open the repository configuration file:
   ```bash
   sudo vim /etc/apt/sources.list.d/kubernetes.list
   ```

2. Update the repository URL to:
   ```bash
   deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /
   ```

3. Save the file and update the package list:
   ```bash
   sudo apt update
   ```

4. Verify available versions:
   ```bash
   sudo apt-cache madison kubeadm
   ```

5. Install `kubeadm` v1.30.0:
   ```bash
   sudo apt-get install kubeadm=1.30.0-1.1
   ```

[Back to Table of Contents](#table-of-contents)

---

### 2. Drain the Control Plane Node (`controlplane`)
Before upgrading the control plane node, it is a good practice to drain it. This ensures that new workloads are not scheduled on the node during the upgrade process.

1. Drain the control plane node:
   ```bash
   kubectl drain controlplane --ignore-daemonsets
   ```

- `--ignore-daemonsets`: Ensures that DaemonSet-managed pods (such as logging or monitoring agents) are not evicted during the drain.
- `--force`: This option can be used to forcefully evict unmanaged pods that may not have graceful shutdown behavior, though this is generally not recommended for production workloads.

[Back to Table of Contents](#table-of-contents)

---

### 3. Upgrade Control Plane Components (`controlplane`)
Once the control plane node has been drained, you can proceed with upgrading the control plane components (API server, Controller Manager, Scheduler).

1. Plan the upgrade:
   ```bash
   sudo kubeadm upgrade plan v1.30.0
   ```

2. Apply the upgrade:
   ```bash
   sudo kubeadm upgrade apply v1.30.0
   ```

This command will upgrade the API server, Controller Manager, and Scheduler to version `v1.30.0`.

[Back to Table of Contents](#table-of-contents)

---

### 4. Upgrade `kubelet` on the Control Plane Node (`controlplane`)
After upgrading the control plane components, upgrade the `kubelet` on the control plane node.

1. Install the new `kubelet` version:
   ```bash
   sudo apt-get install kubelet=1.30.0-1.1
   ```

2. Reload systemd and restart `kubelet`:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl restart kubelet
   ```

Once the `kubelet` is upgraded and restarted, the control plane node is ready.

[Back to Table of Contents](#table-of-contents)

---

### Worker Node (`node01`)

### 5. Upgrade `kubeadm` on the Worker Node (`node01`)
The process of upgrading `kubeadm` on a worker node (`node01`) is similar to that on the control plane node, except that the control plane components are not involved.

1. Open the repository configuration file:
   ```bash
   sudo vim /etc/apt/sources.list.d/kubernetes.list
   ```

2. Update the repository URL:
   ```bash
   deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /
   ```

3. Update the package list:
   ```bash
   sudo apt update
   ```

4. Verify the available versions:
   ```bash
   sudo apt-cache madison kubeadm
   ```

5. Install `kubeadm` v1.30.0:
   ```bash
   sudo apt-get install kubeadm=1.30.0-1.1
   ```

[Back to Table of Contents](#table-of-contents)

---

### 6. Drain the Worker Node (`node01`)
Before upgrading the worker node, drain it to ensure that no workloads are running on the node during the upgrade.

1. Drain the worker node:
   ```bash
   kubectl drain node01 --ignore-daemonsets
   ```

- `--ignore-daemonsets`: Ensures that DaemonSet-managed pods are not disrupted.
- `--force`: This option can be used to forcefully evict unmanaged pods, but it should be used with caution.

[Back to Table of Contents](#table-of-contents)

---

### 7. Upgrade Kubelet on the Worker Node (`node01`)
Once the worker node (`node01`) is drained, upgrade `kubelet`.

1. Run the following to upgrade the node components:
   ```bash
   sudo kubeadm upgrade node
   ```

2. Install the new `kubelet` version:
   ```bash
   sudo apt-get install kubelet=1.30.0-1.1
   ```

3. Reload systemd and restart `kubelet`:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl restart kubelet
   ```

[Back to Table of Contents](#table-of-contents)

---

### 8. Mark the Worker Node as Schedulable
After upgrading the worker node, mark it as schedulable so that workloads can be scheduled on it again.

1. Uncordon the node:
   ```bash
   kubectl uncordon node01
   ```

This makes the node available for scheduling new pods.

[Back to Table of Contents](#table-of-contents)

---

### Final Steps and Verification

### 9. Verification for Both Nodes
Once both nodes are upgraded, verify that they are running Kubernetes version `v1.30.0`.

#### Check Node Version
Use the following command to check the version of both nodes:

```bash
kubectl get nodes
```

**Sample output**:

```bash
NAME           STATUS   ROLES           AGE     VERSION
controlplane   Ready    control-plane   100d    v1.30.0
node01         Ready    <none>          50d     v1.30.0
```

This confirms that both the control plane and worker nodes are running Kubernetes version `v1.30.0`.

#### Check Kubelet Version
On both the control plane node and the worker node, verify the `kubelet` version:

```bash
kubelet --version
```

**Sample output**:

```bash
Kubernetes v1.30.0
```

#### Check Control Plane Version
On the control plane node, verify the control plane component versions:

```bash
kubectl version --short
```

**Sample output**:

```bash
Client Version: v1.30.0
Kubelet Version: v1.30.0
Server Version: v1.30.0
```

[Back to Table of Contents](#table-of-contents)

---

### 10. Additional Notes on `--ignore-daemonsets` and `--force`

- **`--ignore-daemonsets`**: This option is useful when draining nodes. It ensures that **DaemonSet-managed pods** (e.g., monitoring or logging agents) are not evicted during the drain, as these are typically critical services.

- **`--force`**: This flag allows for the forceful eviction of unmanaged pods that may not have proper shutdown behavior. Use this option carefully, especially in production, as it may disrupt services if pods are not terminated gracefully Pods not created by ReplicaContoller, ReplicaSet, Deployment give an error when drained as they can't be recreated. This option allows them to be terminated. 

[Back to

 Table of Contents](#table-of-contents)

---

This concludes the tutorial on upgrading both the **control plane node** (`controlplane`) and the **worker node** (`node01`) to Kubernetes version `v1.30.0`.
