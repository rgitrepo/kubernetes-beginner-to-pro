## **Upgrading and Downgrading Kubernetes Cluster**

### **Table of Contents**
1. [Introduction](#1-introduction)
2. [Preparation Before Upgrading](#2-preparation-before-upgrading)
3. [Upgrade Steps](#3-upgrade-steps)
   - [Step 1: Check Available Versions](#step-1-check-available-versions)
   - [Step 2: Upgrade `kubeadm` on the Control Plane Node](#step-2-upgrade-kubeadm-on-the-control-plane-node)
   - [Step 3: Drain and Cordon the Control Plane Node](#step-3-drain-and-cordon-the-control-plane-node)
   - [Step 4: Upgrade the Control Plane Components](#step-4-upgrade-the-control-plane-components)
   - [Step 5: Upgrade the `kubelet` and `kubectl` on the Control Plane Node](#step-5-upgrade-the-kubelet-and-kubectl-on-the-control-plane-node)
   - [Step 6: Uncordon the Control Plane Node](#step-6-uncordon-the-control-plane-node)
   - [Step 7: Upgrade Worker Nodes](#step-7-upgrade-worker-nodes)
4. [Downgrade Process](#4-downgrade-process)
   - [Step 1: Downgrade `kubeadm`](#step-1-downgrade-kubeadm)
   - [Step 2: Downgrade the Control Plane](#step-2-downgrade-the-control-plane)
   - [Step 3: Downgrade Worker Nodes](#step-3-downgrade-worker-nodes)
5. [Reasons for Upgrading One Node at a Time](#5-reasons-for-upgrading-one-node-at-a-time)
6. [Drain vs Cordon](#6-drain-vs-cordon)
7. [Daemonsets and the `--ignore-daemonsets` Flag](#7-daemonsets-and-the-ignore-daemonsets-flag)
8. [Final Checks](#8-final-checks)
9. [Additional Resources](#9-additional-resources)

---

### **1. Introduction**

Upgrading or downgrading a Kubernetes cluster is a critical task that must be carried out with care to maintain cluster stability, security, and performance. This guide walks you through upgrading your cluster to a new version or downgrading it to a stable one if needed. Along the way, we also cover why upgrades should happen one node at a time, how to handle workloads using **drain** and **cordon**, and the special case of **daemonsets** during node draining.

[Back to TOC](#table-of-contents)

---

### **2. Preparation Before Upgrading**

Before starting the upgrade process, it’s essential to prepare your cluster.

#### **Backup etcd Database**

Always back up the etcd database, which stores the state of your cluster:

```bash
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

#### **Check Cluster Health**

Verify that all nodes and pods are functioning correctly:

```bash
kubectl get nodes
kubectl get pods -A
```

[Back to TOC](#table-of-contents)

---

### **3. Upgrade Steps**

The upgrade process in Kubernetes follows a specific order. First, you upgrade the control plane, then the worker nodes. Always upgrade one node at a time.

#### **Step 1: Check Available Versions**

Use this command to see which versions of `kubeadm` are available for upgrade:

```bash
sudo apt-cache madison kubeadm
```

[Back to TOC](#table-of-contents)

---

#### **Step 2: Upgrade `kubeadm` on the Control Plane Node**

Upgrade the `kubeadm` tool on your control plane node to the desired version:

```bash
sudo apt-get install -y kubeadm=1.30.2-00
```

Verify that the upgrade was successful by running:

```bash
kubeadm version
```

[Back to TOC](#table-of-contents)

---

#### **Step 3: Drain and Cordon the Control Plane Node**

Before upgrading the node, you must cordon and drain it. This ensures no new workloads are scheduled on the node and that existing ones are safely migrated.

1. **Cordon the Node**:
   Cordon makes the node unschedulable, which means no new pods will be scheduled on this node. However, existing pods will continue to run.

   ```bash
   kubectl cordon <control-plane-node>
   ```

   Example output:

   ```bash
   node/control-plane cordoned
   ```

2. **Drain the Node**:
   Draining evicts the existing pods from the node and reschedules them on other available nodes in the cluster.

   ```bash
   kubectl drain <control-plane-node> --ignore-daemonsets --delete-local-data
   ```
   ```bash
   # Command to drain pod that isn't part of ReplicaController, Job, Deployment, Replicaset. Basically that won't come back if deleted.
   kubectl drain <control-plane-node> --ignore-daemonsets --force
   ```
   

   - **`--ignore-daemonsets`**: Daemonset-managed pods will not be evicted during a drain. These pods are typically used for infrastructure services like logging or monitoring and should continue running.
   - **`--delete-local-data`**: This flag allows you to delete local data from the node if necessary, which may include temporary files created by applications.

   Example output:

   ```bash
   node/control-plane already cordoned
   evicting pod my-app-1
   pod/my-app-1 evicted
   ```

   You can check the node's status using:

   ```bash
   kubectl get nodes
   ```

   Example output:

   ```
   NAME            STATUS                      VERSION
   control-plane   Ready,SchedulingDisabled     v1.20.0
   ```

   This shows the node is now cordoned and ready for an upgrade.

[Back to TOC](#table-of-contents)

---

#### **Step 4: Upgrade the Control Plane Components**

Run the following command to upgrade the control plane components, such as the API server, controller manager, and scheduler:

```bash
sudo kubeadm upgrade apply v1.30.2
```

[Back to TOC](#table-of-contents)

---

#### **Step 5: Upgrade the `kubelet` and `kubectl` on the Control Plane Node**

Once the control plane components are upgraded, proceed to upgrade `kubelet` and `kubectl` on the control plane node:

```bash
sudo apt-get install -y kubelet=1.30.2-00 kubectl=1.30.2-00
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

[Back to TOC](#table-of-contents)

---

#### **Step 6: Uncordon the Control Plane Node**

Once the upgrade is complete, uncordon the node to allow new pods to be scheduled on it:

```bash
kubectl uncordon control-plane
```

[Back to TOC](#table-of-contents)

---

#### **Step 7: Upgrade Worker Nodes**

Repeat the same steps (drain, upgrade, and uncordon) for each worker node.

1. Drain the node:

   ```bash
   kubectl drain <worker-node> --ignore-daemonsets --delete-local-data
   ```

2. Upgrade `kubeadm` on the worker node:

   ```bash
   sudo apt-get install -y kubeadm=1.30.2-00
   sudo kubeadm upgrade node
   ```

3. Upgrade `kubelet` and `kubectl`:

   ```bash
   sudo apt-get install -y kubelet=1.30.2-00 kubectl=1.30.2-00
   sudo systemctl daemon-reload
   sudo systemctl restart kubelet
   ```

4. Uncordon the node:

   ```bash
   kubectl uncordon <worker-node>
   ```

Repeat for all worker nodes.

[Back to TOC](#table-of-contents)

---

### **4. Downgrade Process**

In cases where an upgrade introduces issues, you can follow this process to downgrade your cluster.

#### **Step 1: Downgrade `kubeadm`**

Downgrade `kubeadm` to the desired version:

```bash
sudo apt-get install -y kubeadm=1.30.0-00
```

Check the version:

```bash
kubeadm version
```

[Back to TOC](#table-of-contents)

---

#### **Step 2: Downgrade the Control Plane**

Run the following command to downgrade the control plane components:

```bash
sudo kubeadm upgrade apply v1.30.0
```

[Back to TOC](#table-of-contents)

---

#### **Step 3: Downgrade Worker Nodes**

Follow the same steps to downgrade each worker node.

1. Drain the worker node:

   ```bash
   kubectl drain <worker-node> --ignore-daemonsets --delete-local-data
   ```

2. Downgrade `

kubeadm`:

   ```bash
   sudo apt-get install -y kubeadm=1.30.0-00
   sudo kubeadm upgrade node
   ```

3. Downgrade `kubelet` and `kubectl`:

   ```bash
   sudo apt-get install -y kubelet=1.30.0-00 kubectl=1.30.0-00
   sudo systemctl daemon-reload
   sudo systemctl restart kubelet
   ```

4. Uncordon the worker node:

   ```bash
   kubectl uncordon <worker-node>
   ```

Repeat for all worker nodes.

[Back to TOC](#table-of-contents)

---

### **5. Reasons for Upgrading One Node at a Time**

When upgrading a Kubernetes cluster, you must upgrade one node at a time to ensure the cluster remains functional and available. Here’s why:

1. **Application Availability**: By upgrading nodes one at a time, you ensure that workloads are migrated to other nodes, maintaining service availability.

2. **Workload Distribution**: Draining a node shifts its pods to other nodes. If you upgrade multiple nodes simultaneously, you might not have enough resources to handle the load, leading to downtime or performance issues.

3. **Isolating Issues**: Upgrading nodes individually helps isolate issues. If a problem arises, it will only affect one node, not the entire cluster.

4. **Resource Efficiency**: Upgrading one node at a time avoids overwhelming the cluster's resources. Draining multiple nodes simultaneously can lead to resource contention.

[Back to TOC](#table-of-contents)

---

### **6. Drain vs Cordon**

Here’s a quick overview of the differences between draining and cordoning a node:

- **Cordon**: Marks the node as **unschedulable**, meaning no new pods will be scheduled on it. Existing pods remain unaffected and continue running.

  ```bash
  kubectl cordon <node-name>
  ```

- **Drain**: Evicts all running pods from the node and reschedules them on other nodes. This is typically used before upgrading a node or performing maintenance.

  ```bash
  kubectl drain <node-name> --ignore-daemonsets --delete-local-data
  ```

[Back to TOC](#table-of-contents)

---

### **7. Daemonsets and the `--ignore-daemonsets` Flag**

**Daemonsets** ensure that a specific pod is running on every node in a cluster (or a subset of nodes). These are usually used for infrastructure-level workloads, such as logging or monitoring agents, which need to run on every node.

- **Why Use `--ignore-daemonsets`**: When draining a node, the `--ignore-daemonsets` flag tells Kubernetes not to evict pods managed by daemonsets. These pods are essential to the node's infrastructure and should not be rescheduled or removed during maintenance.

```bash
kubectl drain <node-name> --ignore-daemonsets --delete-local-data
```

This command ensures that only application pods are evicted, while infrastructure services running as daemonsets continue to operate.

[Back to TOC](#table-of-contents)

---

### **8. Final Checks**

After upgrading or downgrading your cluster, you should perform the following checks:

1. **Check Node Status**:

   ```bash
   kubectl get nodes
   ```

   Example output after upgrade:

   ```
   NAME           STATUS   VERSION
   control-plane  Ready    v1.30.2
   worker-node-1  Ready    v1.30.2
   worker-node-2  Ready    v1.30.2
   ```

2. **Check Pods and Services**:

   Ensure all pods and services are running smoothly:

   ```bash
   kubectl get pods --all-namespaces
   ```

[Back to TOC](#table-of-contents)

---

### **9. Additional Resources**

- [Kubernetes Documentation: Upgrading kubeadm Clusters](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
- [Draining a Node](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/)
- [Daemonsets in Kubernetes](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)

