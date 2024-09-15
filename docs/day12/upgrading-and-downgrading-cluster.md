
## **Upgrading and Downgrading Kubernetes Cluster: A Comprehensive Tutorial**

### **Table of Contents (TOC)**
1. [Introduction](#introduction)
2. [Kubernetes Versioning](#kubernetes-versioning)
3. [Why Upgrade or Downgrade?](#why-upgrade-or-downgrade)
4. [Preparation Before Upgrading/Downgrading](#preparation-before-upgradingdowngrading)
5. [Upgrade Process](#upgrade-process)
   - [1. Upgrading Using Cloud Providers](#1-upgrading-using-cloud-providers)
   - [2. Upgrading Using kubeadm](#2-upgrading-using-kubeadm)
6. [Downgrade Process](#downgrade-process)
7. [Important Concepts](#important-concepts)
   - [1. Drain and Cordon](#1-drain-and-cordon)
   - [2. Kubernetes Node States](#2-kubernetes-node-states)
   - [3. Supported Versions](#3-supported-versions)
8. [Common Commands and Outputs](#common-commands-and-outputs)
9. [Additional Resources](#additional-resources)

---

### **1. Introduction**
Upgrading or downgrading a Kubernetes cluster ensures that the cluster stays up-to-date with the latest features, security patches, and improvements. Similarly, in certain cases, downgrading might be necessary to revert to a more stable version if issues arise. This tutorial provides a clear guide on how to perform upgrades and downgrades using both cloud-based and `kubeadm`-based clusters.

[Back to TOC](#table-of-contents-toc)

---

### **2. Kubernetes Versioning**
Kubernetes follows a semantic versioning scheme: `vA.B.C`. Here's what each part means:
- **A**: Major version. It is rarely changed; Kubernetes has been on version 1.x for a long time.
- **B**: Minor version. It changes frequently (around every 3-4 months).
- **C**: Patch version. Patches are for bug fixes, performance improvements, and security fixes.

For example, Kubernetes version `v1.30.2` means:
- **1**: Major version
- **30**: Minor version
- **2**: Patch version

Each year, Kubernetes releases two minor versions, and only the last four minor versions are supported for backward compatibility.

[Back to TOC](#table-of-contents-toc)

---

### **3. Why Upgrade or Downgrade?**
#### **Reasons to Upgrade:**
- **Security Fixes**: Newer versions contain essential patches.
- **New Features**: Minor versions introduce new functionalities.
- **Improved Performance**: Upgrades often come with optimizations.
  
#### **Reasons to Downgrade:**
- **Stability Issues**: Sometimes, newer versions may introduce instability in certain workloads.
- **Feature Deprecation**: If a feature crucial to your operation has been deprecated.

[Back to TOC](#table-of-contents-toc)

---

### **4. Preparation Before Upgrading/Downgrading**
1. **Backup Important Data**: Ensure that etcd and other critical components are backed up.
2. **Check Compatibility**: Ensure that you are only upgrading or downgrading to supported versions (n-1, n+1 compatibility rule).
3. **Test in Non-Production**: Test the process in a staging or non-production environment.
4. **Monitor the Cluster**: Enable monitoring tools to observe cluster behavior before, during, and after the upgrade.

[Back to TOC](#table-of-contents-toc)

---

### **5. Upgrade Process**
#### **1. Upgrading Using Cloud Providers**
Cloud providers such as GKE, EKS, or AKS make the upgrade process seamless:
1. Navigate to the **Cluster Management Dashboard**.
2. Click the **Upgrade** button.
3. Choose the target Kubernetes version, and the cluster will be upgraded with minimal downtime.

#### Example:
For GKE (Google Kubernetes Engine), upgrading is often just a button click.

1. Log in to your GCP console.
2. Go to the **Kubernetes Engine** page.
3. Choose the cluster you want to upgrade.
4. Click **Upgrade** and select the latest stable version.

```bash
# For GKE, you can also use gcloud command:
gcloud container clusters upgrade <CLUSTER_NAME> --master --cluster-version=<NEW_VERSION>
```

---

#### **2. Upgrading Using `kubeadm`**
For clusters created with `kubeadm`, the process is more involved:

**Step-by-Step Process for `kubeadm`:**
1. **Check Available Versions:**
   Use the following command to check which Kubernetes versions are available:
   ```bash
   sudo apt-cache madison kubeadm | tac
   ```
In the Linux operating system, the tac command is used to concatenate and print the files in reverse format. When the input or specified file is not provided to the command then the tac command will read the standard input. The tac command is exactly opposite to the “cat” command.

2. **Upgrade `kubeadm` Tool:**
   Upgrade `kubeadm` before upgrading the cluster:
   ```bash
   sudo apt-get update && sudo apt-get install -y kubeadm=1.30.2-00
   ```

3. **Upgrade the Control Plane:**
   Run the upgrade command for the control plane:
   ```bash
   sudo kubeadm upgrade plan
   sudo kubeadm upgrade apply v1.30.2
   ```

4. **Upgrade the `kubelet` and `kubectl`:**
   After upgrading the control plane, you need to upgrade the kubelet and kubectl on each node:
   ```bash
   sudo apt-get install -y kubelet=1.30.2-00 kubectl=1.30.2-00
   sudo systemctl daemon-reload
   sudo systemctl restart kubelet
   ```

5. **Drain and Cordon Nodes:**
   Ensure the nodes are properly drained before upgrade:
   ```bash
   kubectl drain <NODE_NAME> --ignore-daemonsets
   ```

6. **Uncordon the Nodes:**
   Once upgraded, uncordon the nodes to allow scheduling:
   ```bash
   kubectl uncordon <NODE_NAME>
   ```

[Back to TOC](#table-of-contents-toc)

---

### **6. Downgrade Process**
The downgrade process is similar to the upgrade process, but it must be done carefully to avoid incompatibilities.

**Steps to Downgrade:**
1. **Drain and Cordon the Nodes**: 
   Just like in upgrades, ensure that no pods are scheduled on the node being downgraded.
   
   ```bash
   kubectl drain <NODE_NAME> --ignore-daemonsets
   ```

2. **Install the Previous Version**:
   Install the desired (older) version of `kubeadm`, `kubelet`, and `kubectl`:
   ```bash
   sudo apt-get install -y kubeadm=<OLD_VERSION> kubelet=<OLD_VERSION> kubectl=<OLD_VERSION>
   ```

3. **Apply Downgrade:**
   ```bash
   sudo kubeadm upgrade apply <OLD_VERSION>
   ```

4. **Restart `kubelet` and Uncordon the Node**:
   ```bash
   sudo systemctl restart kubelet
   kubectl uncordon <NODE_NAME>
   ```

[Back to TOC](#table-of-contents-toc)

---

### **7. Important Concepts**
#### **1. Drain and Cordon**
- **Drain**: Moves all pods from a node to other nodes in the cluster.
- **Cordon**: Marks the node as unschedulable, preventing new pods from being scheduled there.

```bash
# Example of draining a node
kubectl drain <NODE_NAME> --ignore-daemonsets

# Example of cordoning a node
kubectl cordon <NODE_NAME>
```

#### **2. Kubernetes Node States**
- **Ready**: The node is ready to accept and run pods.
- **NotReady**: The node is unavailable for running pods.
- **SchedulingDisabled**: The node is cordoned and won’t accept new pods.

#### **3. Supported Versions**
Kubernetes allows upgrades and downgrades within a two-version window. You can upgrade from `n-1` to `n+1`, but not more than two versions apart.

[Back to TOC](#table-of-contents-toc)

---

### **8. Common Commands and Outputs**
Here are some frequently used commands during upgrades or downgrades, along with their expected outputs:

- **Check Current Version**:
   ```bash
   kubectl version --short
   ```
   Output:
   ```
   Client Version: v1.30.0
   Server Version: v1.30.0
   ```

- **List Available Versions for `kubeadm`**:
   ```bash
   sudo apt-cache madison kubeadm
   ```

- **Upgrade Plan for `kubeadm`**:
   ```bash
   sudo kubeadm upgrade plan
   ```
   Output:
   ```
   Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
   COMPONENT   CURRENT   AVAILABLE
   kubelet     1.30.0    1.30.2
   ```

- **Upgrade `kubeadm`**:
   ```bash
   sudo kubeadm upgrade apply v1.30.2
   ```

[Back to TOC](#table-of-contents-toc)

---

### **9. Additional Resources**
- [Kubernetes Official Upgrade Documentation](https://kubernetes.io/docs/tasks/administer-cluster/cluster-upgrade/)
- [Kube

adm Upgrade Docs](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)

[Back to TOC](#table-of-contents-toc)

