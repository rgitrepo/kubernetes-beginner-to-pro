### Tutorial: Identifying if `kubeadm` Was Used and How Static Pods Are Managed in Kubernetes

---

### **Table of Contents**

1. [Introduction](#1-introduction)  
2. [Checking for `kubeadm` Usage](#2-checking-for-kubeadm-usage)  
   - [Inspecting `/etc/kubernetes`](#21-inspecting-etckubernetes)  
   - [Inspecting `kubeadm` Configuration](#22-inspecting-kubeadm-configuration)  
   - [Looking for `kubeadm`-specific Annotations](#23-looking-for-kubeadm-specific-annotations)  
3. [Static Pods in `/etc/kubernetes/manifests`](#3-static-pods-in-etckubernetesmanifests)  
   - [Checking the Manifests Folder](#31-checking-the-manifests-folder)  
   - [Inspecting a Static Pod Manifest](#32-inspecting-a-static-pod-manifest)  
4. [Clusters Set Up the "Hard Way" Using `systemd`](#4-clusters-set-up-the-hard-way-using-systemd)  
   - [Checking for `systemd` Services](#41-checking-for-systemd-services)  
   - [Inspecting `systemd` Service Files](#42-inspecting-systemd-service-files)  
   - [Checking `kubelet` Arguments](#43-checking-kubelet-arguments)  
5. [Conclusion](#5-conclusion)  

---

### **1. Introduction**

Kubernetes clusters can be set up in various ways, including using tools like `kubeadm`, or manually managing components using `systemd`. This tutorial explains how to determine if a cluster was set up using `kubeadm` and whether static pods are managed via the `/etc/kubernetes/manifests` directory or configured manually with `systemd`.

[Back to TOC](#table-of-contents)

---

### **2. Checking for `kubeadm` Usage**

#### **2.1 Inspecting `/etc/kubernetes`**

`kubeadm` creates specific directories and configuration files in `/etc/kubernetes`. Check for their presence:

```bash
ls /etc/kubernetes
```

If you find files like `admin.conf`, `kubelet.conf`, `pki/` (for certificates), or `manifests/`, the cluster was likely set up using `kubeadm`.

#### Key Files to Look For:
- **`kubeadm-config.yaml`**: Contains details about the cluster configuration.
- **`manifests/`**: Houses YAML files for static pods.
- **`pki/`**: Contains cluster certificates.

[Back to TOC](#table-of-contents)

---

#### **2.2 Inspecting `kubeadm` Configuration**

If the file `kubeadm-config.yaml` exists, inspect it to confirm the cluster setup:

```bash
cat /etc/kubernetes/kubeadm-config.yaml
```

This file often contains API server details, controller-manager settings, and other configurations specific to `kubeadm`.

[Back to TOC](#table-of-contents)

---

#### **2.3 Looking for `kubeadm`-specific Annotations**

`kubeadm` adds annotations to nodes in the `kube-system` namespace. Check for them:

```bash
kubectl get nodes -o jsonpath="{.items[*].metadata.annotations}" | grep kubeadm
```

If the output includes annotations related to `kubeadm`, the cluster was initialized using `kubeadm`.

[Back to TOC](#table-of-contents)

---

### **3. Static Pods in `/etc/kubernetes/manifests`**

#### **3.1 Checking the Manifests Folder**

`kubeadm` sets up static pods for core components in `/etc/kubernetes/manifests`. List the contents:

```bash
ls /etc/kubernetes/manifests
```

Files such as `kube-apiserver.yaml`, `kube-controller-manager.yaml`, and `kube-scheduler.yaml` indicate that static pods are used.

[Back to TOC](#table-of-contents)

---

#### **3.2 Inspecting a Static Pod Manifest**

To verify the configuration, inspect one of the static pod manifests:

```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

These YAML files define the configuration for Kubernetes core components and are managed by the kubelet.

[Back to TOC](#table-of-contents)

---

### **4. Clusters Set Up the "Hard Way" Using `systemd`**

Clusters not set up with `kubeadm` might use `systemd` to manage Kubernetes components.

#### **4.1 Checking for `systemd` Services**

List active services to identify Kubernetes components managed by `systemd`:

```bash
systemctl list-units | grep kube
```

If services like `kube-apiserver.service` or `kube-controller-manager.service` are listed, the cluster was likely set up manually.

[Back to TOC](#table-of-contents)

---

#### **4.2 Inspecting `systemd` Service Files**

Review the service files for details about the configuration:

```bash
cat /etc/systemd/system/kube-apiserver.service
```

These files contain details such as command-line arguments and configuration paths for the components.

[Back to TOC](#table-of-contents)

---

#### **4.3 Checking `kubelet` Arguments**

Inspect the `kubelet` service file to determine how it is configured:

```bash
cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

- **With `kubeadm`:** Arguments like `--pod-manifest-path=/etc/kubernetes/manifests` indicate static pod management.
- **Without `kubeadm`:** This argument might be absent, and the kubelet might use other configurations.

[Back to TOC](#table-of-contents)

---

### **5. Conclusion**

By following this tutorial, you can determine whether your Kubernetes cluster was set up using `kubeadm` or manually with `systemd`. Additionally, you can identify whether static pods are being used and how they are managed.

[Back to TOC](#table-of-contents)  

