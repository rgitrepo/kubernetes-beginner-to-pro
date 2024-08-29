# Quality of Service (QoS) in Kubernetes

## Table of Contents

1. [Introduction to Quality of Service (QoS)](#introduction-to-quality-of-service-qos)
2. [Understanding Eviction in Kubernetes](#understanding-eviction-in-kubernetes)
3. [QoS Classes in Kubernetes](#qos-classes-in-kubernetes)
   - [Guaranteed](#guaranteed)
   - [Burstable](#burstable)
   - [Best Effort](#best-effort)
4. [Example YAML Configuration for QoS](#example-yaml-configuration-for-qos)
5. [Conclusion](#conclusion)

---

## Introduction to Quality of Service (QoS)

[Back to TOC](#table-of-contents)

In Kubernetes, Quality of Service (QoS) guarantees the stability of your pods by ensuring they receive the necessary resources, such as CPU and memory. Kubernetes assigns a QoS class to each pod, which helps manage resource allocation, especially under conditions where resources are scarce. Understanding QoS is crucial for maintaining the reliability and stability of your applications in a Kubernetes cluster.

## Understanding Eviction in Kubernetes

[Back to TOC](#table-of-contents)

Eviction in Kubernetes is a process where pods are terminated to free up resources for higher-priority workloads. When the cluster faces resource constraints, Kubernetes may evict lower-priority pods to ensure that more critical pods continue to run smoothly. The QoS class assigned to a pod influences its likelihood of being evicted.

### How Eviction Works

When the scheduler detects that resources like CPU and memory are low, it prioritizes pods based on their QoS class. If necessary, it may instruct the kubelet to evict pods that do not have a high-priority QoS class, ensuring that essential services remain operational.

## QoS Classes in Kubernetes

[Back to TOC](#table-of-contents)

Kubernetes defines three QoS classes: **Guaranteed**, **Burstable**, and **Best Effort**. These classes determine the priority of the pods and their protection from eviction.

### Guaranteed

[Back to TOC](#table-of-contents)

A pod is classified under the **Guaranteed** QoS class when the resource requests (CPU and memory) and limits are both set and are equal for all containers within the pod. This class provides the highest level of protection from eviction. Even if the cluster is under heavy load, pods in the Guaranteed class are the least likely to be evicted.

**Example:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: guaranteed-pod
spec:
  containers:
  - name: my-container
    image: nginx
    resources:
      requests:
        memory: "1Gi"
        cpu: "500m"
      limits:
        memory: "1Gi"
        cpu: "500m"
```

### Burstable

[Back to TOC](#table-of-contents)

A pod falls under the **Burstable** QoS class when it has resource requests and limits defined, but they are not equal. This class offers moderate protection from eviction. While it is more protected than Best Effort, it is still more susceptible to eviction compared to the Guaranteed class.

**Example:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: burstable-pod
spec:
  containers:
  - name: my-container
    image: nginx
    resources:
      requests:
        memory: "500Mi"
        cpu: "250m"
      limits:
        memory: "1Gi"
        cpu: "500m"
```

### Best Effort

[Back to TOC](#table-of-contents)

Pods in the **Best Effort** QoS class do not have any resource requests or limits set. This is the lowest priority class, meaning these pods are the first to be evicted when resources are constrained.

**Example:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: best-effort-pod
spec:
  containers:
  - name: my-container
    image: nginx
```

## Example YAML Configuration for QoS

[Back to TOC](#table-of-contents)

To understand how QoS classes are assigned, let's consider the following YAML configuration for different types of pods. These examples show how the Kubernetes scheduler categorizes pods into different QoS classes based on the resources specified.

1. **Guaranteed QoS Example:**

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: guaranteed-pod
   spec:
     containers:
     - name: my-container
       image: nginx
       resources:
         requests:
           memory: "1Gi"
           cpu: "500m"
         limits:
           memory: "1Gi"
           cpu: "500m"
   ```

2. **Burstable QoS Example:**

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: burstable-pod
   spec:
     containers:
     - name: my-container
       image: nginx
       resources:
         requests:
           memory: "500Mi"
           cpu: "250m"
         limits:
           memory: "1Gi"
           cpu: "500m"
   ```

3. **Best Effort QoS Example:**

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: best-effort-pod
   spec:
     containers:
     - name: my-container
       image: nginx
   ```

## Conclusion

[Back to TOC](#table-of-contents)

Understanding Kubernetes QoS classes is essential for effectively managing resource allocation and ensuring the stability of your applications. By correctly configuring the resources in your pod specifications, you can influence which QoS class your pods fall under and ensure that critical workloads are protected from eviction even under resource constraints.
