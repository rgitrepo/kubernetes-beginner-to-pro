Here is the updated Table of Contents (TOC) with working links, ensuring clarity and logical flow for the tutorial on upgrading and downgrading a Kubernetes cluster.

---

## **Upgrading and Downgrading Kubernetes Cluster: Step-by-Step Guide**

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
6. [Commands and Outputs](#6-commands-and-outputs)
7. [Final Checks](#7-final-checks)
8. [Additional Resources](#8-additional-resources)

---

### **1. Introduction**

This guide provides a comprehensive walkthrough on upgrading and **downgrading** a Kubernetes cluster using `kubeadm`. Both processes are critical in cluster management to either benefit from newer features or revert to previous versions if issues arise. It also explains why **upgrading one node at a time** is essential for maintaining cluster health.

[Back to TOC](#table-of-contents)

---

### **2. Preparation Before Upgrading**

Before you upgrade or downgrade a Kubernetes cluster, follow the preparation steps to ensure cluster stability and data integrity.

#### **Backup etcd**:
Always back up your etcd database:

```bash
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

#### **Check Cluster Health**:
Verify the status of the nodes and pods:

```bash
kubectl get nodes
kubectl get pods -A
```

[Back to TOC](#table-of-contents)

---

### **3. Upgrade Steps**

#### **Step 1: Check Available Versions**

```bash
sudo apt-cache madison kubeadm
```

[Back to TOC](#table-of-contents)

---

#### **Step 2: Upgrade `kubeadm` on the Control Plane Node**

Upgrade `kubeadm` to the desired version:

```bash
sudo apt-get install -y kubeadm=1.30.2-00
```

[Back to TOC](#table-of-contents)

---

#### **Step 3: Drain and Cordon the Control Plane Node**

Draining and cordoning the node will prevent new pods from being scheduled and safely migrate workloads.

```bash
kubectl drain control-plane --ignore-daemonsets
```

Verify node status:

```bash
kubectl get nodes
```

[Back to TOC](#table-of-contents)

---

#### **Step 4: Upgrade the Control Plane Components**

Apply the upgrade to the control plane components:

```bash
sudo kubeadm upgrade apply v1.30.2
```

[Back to TOC](#table-of-contents)

---

#### **Step 5: Upgrade the `kubelet` and `kubectl` on the Control Plane Node**

Upgrade `kubelet` and `kubectl`:

```bash
sudo apt-get install -y kubelet=1.30.2-00 kubectl=1.30.2-00
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

[Back to TOC](#table-of-contents)

---

#### **Step 6: Uncordon the Control Plane Node**

Uncordon the control plane node:

```bash
kubectl uncordon control-plane
```

[Back to TOC](#table-of-contents)

---

#### **Step 7: Upgrade Worker Nodes**

Repeat the same steps (drain, upgrade, and uncordon) for each worker node.

[Back to TOC](#table-of-contents)

---

### **4. Downgrade Process**

If the upgrade introduces bugs or performance issues, downgrading to a stable version may be necessary.

---

#### **Step 1: Downgrade `kubeadm`**

Downgrade `kubeadm` to a previous version:

```bash
sudo apt-get install -y kubeadm=1.30.0-00
```

Check if the downgrade was successful:

```bash
kubeadm version
```

[Back to TOC](#table-of-contents)

---

#### **Step 2: Downgrade the Control Plane**

Downgrade the control plane components:

```bash
sudo kubeadm upgrade apply v1.30.0
```

[Back to TOC](#table-of-contents)

---

#### **Step 3: Downgrade Worker Nodes**

After downgrading the control plane, repeat the same process for each worker node:

```bash
kubectl drain node-01 --ignore-daemonsets
sudo apt-get install -y kubeadm=1.30.0-00
sudo kubeadm upgrade node
sudo apt-get install -y kubelet=1.30.0-00 kubectl=1.30.0-00
sudo systemctl daemon-reload
sudo systemctl restart kubelet
kubectl uncordon node-01
```

[Back to TOC](#table-of-contents)

---

### **5. Reasons for Upgrading One Node at a Time**

When upgrading a Kubernetes cluster, it’s crucial to upgrade only one node at a time to minimize disruption and prevent potential failures. Here's why:

#### **1. Maintaining Application Availability**
By upgrading nodes one at a time, you ensure that your applications continue running on other available nodes. The pods running on the node being upgraded are drained and rescheduled on other nodes, ensuring that your application is still available and avoiding downtime.

#### **2. Workload Distribution and Scheduling**
When you drain a node, Kubernetes shifts its workload to other available nodes. If you try to upgrade multiple nodes simultaneously, there may not be enough resources on the remaining nodes to handle the increased load, leading to performance issues or potential downtime.

#### **3. Isolating Potential Issues**
Upgrading nodes individually allows you to isolate any potential issues that may arise during the upgrade process. If an error occurs during an upgrade, it will only impact the node being upgraded, leaving the rest of the cluster unaffected. This step-by-step approach gives you more control over the upgrade and helps prevent cluster-wide disruptions.

#### **4. Resource Efficiency**
By upgrading one node at a time, you avoid overloading the cluster’s resources. Upgrading multiple nodes simultaneously can lead to resource contention and performance degradation as the cluster’s available capacity temporarily decreases during the process.

[Back to TOC](#table-of-contents)

---

### **6. Commands and Outputs**

After each step in the upgrade or downgrade process, use the following commands to verify the status of your nodes:

- **Get Nodes:**
  
  ```bash
  kubectl get nodes
  ```

  Example output before the upgrade/downgrade:

  ```
  NAME           STATUS                     VERSION
  control-plane  Ready,SchedulingDisabled   v1.20.0
  worker-node-1  Ready                      v1.20.0
  worker-node-2  Ready                      v1.20.0
  ```

  Example output after the upgrade/downgrade:

  ```
  NAME           STATUS                     VERSION
  control-plane  Ready                      v1.21.0
  worker-node-1  Ready                      v1.21.0
  worker-node-2  Ready                      v1.21.0
  ```

- **Check Pods and Services:**

  Verify that all the pods and services are running as expected by using:

  ```bash
  kubectl get pods --all-namespaces
  ```

[Back to TOC](#table-of-contents)

---

### **7. Final Checks**

After completing the upgrade or downgrade process for all nodes, perform these final checks to ensure the cluster is healthy:

1. **Check Node Status:**

   Run `kubectl get nodes` to check the status of all nodes in the cluster:

   ```bash
   kubectl get nodes
   ```

2. **Check Pods and Services:**

   Ensure that there are no pods

 in a `CrashLoopBackOff` state or pending state by running:

   ```bash
   kubectl get pods --all-namespaces
   ```

[Back to TOC](#table-of-contents)

---

### **8. Additional Resources**

- [Kubernetes Documentation: Upgrading kubeadm Clusters](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
- [Kubernetes - Draining Nodes](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/)
- [Downgrading Kubernetes Cluster](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#downgrading-a-cluster)

[Back to TOC](#table-of-contents)

