## **Kubernetes Security Context and Capabilities: Comprehensive Guide**

### **Table of Contents**

1. [Introduction](#1-introduction)
2. [Understanding Security Context](#2-understanding-security-context)
   - [1. What is SecurityContext?](#what-is-securitycontext)
   - [2. Configuring ReadOnlyRootFilesystem](#configuring-readonlyrootfilesystem)
   - [3. AllowPrivilegeEscalation](#allowprivilegeescalation)
3. [Security Context at Pod and Container Levels](#3-security-context-at-pod-and-container-levels)
   - [1. Pod-level SecurityContext](#pod-level-securitycontext)
   - [2. Container-level SecurityContext](#container-level-securitycontext)
4. [Hands-On Examples](#4-hands-on-examples)
   - [1. Create a Secure Pod](#create-a-secure-pod)
   - [2. Demonstrate Pod Without Security](#demonstrate-pod-without-security)
5. [Capabilities in Kubernetes](#5-capabilities-in-kubernetes)
   - [1. What are Capabilities?](#what-are-capabilities)
   - [2. Adding and Removing Capabilities](#adding-and-removing-capabilities)
6. [SecurityContext and Capabilities Demo](#6-securitycontext-and-capabilities-demo)
   - [1. Example with NET_ADMIN](#example-with-net_admin)
   - [2. Using SECURE COMPUTING (seccomp)](#using-secure-computing-seccomp)
7. [Conclusion](#7-conclusion)
8. [Additional Resources](#8-additional-resources)

---

### **1. Introduction**

Kubernetes provides several mechanisms to ensure the security of your pods and containers. Two of the most important features for security are **SecurityContext** and **Capabilities**. These allow you to control how a pod behaves, which user it runs as, and what permissions it has inside the container.

In this tutorial, we will go step by step through configuring these security features and understanding how they can protect your containers from malicious activities.

[Back to TOC](#table-of-contents)

---

### **2. Understanding Security Context**

#### **1. What is SecurityContext?**

**SecurityContext** is a Kubernetes feature that allows you to define security settings for both containers and pods. It helps in controlling permissions, user privileges, and other security configurations that enhance the protection of your containers.

Example fields of SecurityContext:
- **runAsUser**: Specifies the user ID to run the container.
- **allowPrivilegeEscalation**: Prevents privilege escalation (if set to `false`).
- **readOnlyRootFilesystem**: Ensures the root filesystem is mounted as read-only.

Let’s explore two critical fields in the **SecurityContext**:

#### **2. Configuring ReadOnlyRootFilesystem**

When the root filesystem is writable, any process inside the container can modify critical files, which opens up security risks, such as file tampering. Setting `readOnlyRootFilesystem` to `true` prevents this.

```yaml
securityContext:
  readOnlyRootFilesystem: true
```

By making the filesystem read-only, you ensure that no process can make changes to the container’s root files, safeguarding the container against a **container breakout attack**.

#### **3. AllowPrivilegeEscalation**

By default, containers in Kubernetes have administrator (root) privileges. **allowPrivilegeEscalation** ensures that the container cannot gain more privileges than it was started with.

```yaml
securityContext:
  allowPrivilegeEscalation: false
```

Setting this field to `false` prevents the container from escalating privileges, which is crucial in environments where you want to ensure containers run with the least privilege possible.

[Back to TOC](#table-of-contents)

---

### **3. Security Context at Pod and Container Levels**

#### **1. Pod-level SecurityContext**

SecurityContext can be applied at the **pod level** to enforce security policies across all containers within the pod. When defined at this level, the security policies apply to all containers.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsUser: 1000
    readOnlyRootFilesystem: true
    allowPrivilegeEscalation: false
  containers:
  - name: secure-container
    image: nginx
```

In this example, **runAsUser: 1000** ensures the pod does not run as root.

#### **2. Container-level SecurityContext**

SecurityContext can also be applied at the **container level** to define security configurations for each container independently.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  containers:
  - name: secure-container
    image: nginx
    securityContext:
      runAsUser: 1000
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
```

Here, the security settings apply only to the container **secure-container**.

[Back to TOC](#table-of-contents)

---

### **4. Hands-On Examples**

#### **1. Create a Secure Pod**

Let’s create a pod with a **read-only root filesystem** and **privilege escalation disabled**. The pod will run a **nginx** container:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
    securityContext:
      readOnlyRootFilesystem: true
      allowPrivilegeEscalation: false
```

Apply the configuration:

```bash
kubectl apply -f secure-pod.yaml
```

Verify the pod is running:

```bash
kubectl get pods
```

Now, try interacting with the container:

```bash
kubectl exec -it secure-pod -- bash
```

Attempt to modify the filesystem:

```bash
rm -rf /etc
```

You should see an error message like:

```bash
rm: cannot remove '/etc': Read-only file system
```

This confirms that **readOnlyRootFilesystem** is successfully preventing any modification to the filesystem.

#### **2. Demonstrate Pod Without Security**

Next, let’s create a pod without the security context and observe the difference:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: insecure-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
```

Apply the configuration:

```bash
kubectl apply -f insecure-pod.yaml
```

Now, try modifying the filesystem:

```bash
kubectl exec -it insecure-pod -- bash
rm -rf /etc
```

Unlike the secure pod, you will find that the **insecure-pod** allows this action without restrictions.

[Back to TOC](#table-of-contents)

---

### **5. Capabilities in Kubernetes**

#### **1. What are Capabilities?**

**Capabilities** in Linux allow fine-grained control over system privileges. Instead of granting full root access to containers, you can add or remove specific capabilities. For example, instead of running a container as root, you can give it only the **CAP_NET_ADMIN** capability to manage network settings.

#### **2. Adding and Removing Capabilities**

You can define **add** or **drop** capabilities in the **securityContext**. Let’s explore how:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: capability-pod
spec:
  containers:
  - name: privileged-container
    image: nginx
    securityContext:
      capabilities:
        add: ["NET_ADMIN"]
        drop: ["ALL"]
```

In this example, we **add** the `NET_ADMIN` capability to allow network administration and **drop** all other capabilities. 

[Back to TOC](#table-of-contents)

---

### **6. SecurityContext and Capabilities Demo**

#### **1. Example with NET_ADMIN**

Let’s create a pod with the **NET_ADMIN** capability:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: netadmin-pod
spec:
  containers:
  - name: net-container
    image: busybox
    command: ["sleep", "3600"]
    securityContext:
      capabilities:
        add: ["NET_ADMIN"]
```

Apply the configuration:

```bash
kubectl apply -f netadmin-pod.yaml
```

Now, exec into the pod:

```bash
kubectl exec -it netadmin-pod -- bash
```

You can now run networking commands like:

```bash
ip link add dummy0 type dummy
```

Without the **NET_ADMIN** capability, this command would fail.

#### **2. Using SECURE COMPUTING (seccomp)**

**seccomp** is a Linux kernel feature that restricts system calls that a process can make. It can be configured in Kubernetes to further secure your containers by limiting the available system calls.

To apply **seccomp**, use the following SecurityContext:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: seccomp-pod
spec:
  containers:
  - name: nginx
    image: nginx
    securityContext:
      seccompProfile:
        type: RuntimeDefault
```

This applies a **RuntimeDefault** seccomp profile, which is the default secure computing profile defined by Kubernetes.



[Back to TOC](#table-of-contents)

---

### **7. Conclusion**

**SecurityContext** and **Capabilities** in Kubernetes offer powerful tools to secure your containers by enforcing the principle of least privilege. By controlling the privileges a container has, and limiting what it can do, you minimize the risk of security breaches. These features should be an essential part of any production-grade Kubernetes deployment.

**Capabilities** allow for fine-tuned control of what a container can do without giving it full root access, and **SecurityContext** can prevent privilege escalation and limit write access to the filesystem.

[Back to TOC](#table-of-contents)

---

### **8. Additional Resources**

- [Kubernetes SecurityContext Documentation](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
- [Linux Capabilities Documentation](http://man7.org/linux/man-pages/man7/capabilities.7.html)
- [Falco for Runtime Security](https://falco.org/)

