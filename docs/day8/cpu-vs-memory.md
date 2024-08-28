
## Tutorial: Understanding CPU vs Memory in Kubernetes

### Table of Contents
1. [Introduction](#introduction)
2. [What is CPU in Kubernetes?](#what-is-cpu-in-kubernetes)
    - [Compressible vs. Non-Compressible Resources](#compressible-vs-non-compressible-resources)
    - [Understanding CPU Allocation in Kubernetes](#understanding-cpu-allocation-in-kubernetes)
    - [Example: Defining CPU Requests and Limits](#example-defining-cpu-requests-and-limits)
3. [What is Memory in Kubernetes?](#what-is-memory-in-kubernetes)
    - [Memory Allocation and Non-Compressibility](#memory-allocation-and-non-compressibility)
    - [Example: Defining Memory Requests and Limits](#example-defining-memory-requests-and-limits)
4. [Dangers of Not Setting Resource Limits](#dangers-of-not-setting-resource-limits)
5. [Comparing CPU and Memory](#comparing-cpu-and-memory)
    - [Key Differences](#key-differences)
6. [Practical Example](#practical-example)
    - [Creating a Pod with Defined Resources](#creating-a-pod-with-defined-resources)
    - [Viewing Resource Allocation](#viewing-resource-allocation)
7. [Conclusion](#conclusion)

---

### 1. Introduction
In Kubernetes, managing resources such as CPU and Memory is critical for optimizing the performance and efficiency of your applications. Understanding how Kubernetes allocates and manages these resources is key to ensuring that your applications run smoothly.

### 2. What is CPU in Kubernetes?

#### Compressible vs. Non-Compressible Resources
CPU is a **compressible resource**. This means that it can be allocated in varying amounts based on availability. If a process needs more CPU, it can potentially use more, but it’s also possible to limit the CPU usage of a process.

- **Compressible Resource (CPU)**: You can adjust and allocate CPU usage based on needs. For example, if you’re running a game and it starts to lag, it usually means your CPU is slow or its threads are fully utilized. In contrast, if a game crashes, it’s likely because your memory (RAM) is full.

Memory, on the other hand, is a **non-compressible resource**. Once it’s allocated, it cannot be taken back or compressed.

#### Understanding CPU Allocation in Kubernetes
In Kubernetes, CPU is measured in **millicores** (m). The concept of millicores allows for precise allocation of CPU resources. 

- **1 CPU Core = 1000 millicores (m)**.
- When you see `100m` CPU defined in a YAML file, it means the pod is allocated `100` millicores, or 0.1 of a CPU core.
- A **virtual core** (vCore) in the cloud is typically equivalent to half of a real physical core. For example, 1 real core equals 2 vCores in the cloud.

#### Example: Defining CPU Requests and Limits
In Kubernetes, you can define both CPU requests and CPU limits for your pods:

```yaml
resources:
  requests:
    cpu: "100m"
  limits:
    cpu: "500m"
```

- **Requests**: The amount of CPU that is guaranteed to the pod.
- **Limits**: The maximum amount of CPU that the pod is allowed to use.

### 3. What is Memory in Kubernetes?

#### Memory Allocation and Non-Compressibility
Memory in Kubernetes is a **non-compressible resource**. Once memory is allocated to a pod, it cannot be reclaimed or adjusted.

- **Non-Compressible Resource (Memory)**: If a pod is allocated 1000 MB of memory, and it uses it all, you cannot reclaim it. This means memory allocation needs to be managed carefully to avoid overconsumption or waste.

Memory is typically measured in **bytes**, with common units being:

- **MB** (Megabytes)
- **GB** (Gigabytes)

However, in Kubernetes, you might also see memory measured in **MiB** (Mebibytes), **GiB** (Gibibytes), etc. 

- **1 MiB = 1.048576 MB**. This difference becomes significant when dealing with large quantities of data.

#### Example: Defining Memory Requests and Limits
Similar to CPU, you can define memory requests and limits in your Kubernetes YAML files:

```yaml
resources:
  requests:
    memory: "256Mi"
  limits:
    memory: "512Mi"
```

- **Requests**: The amount of memory guaranteed to the pod.
- **Limits**: The maximum memory the pod is allowed to use.

### 4. Dangers of Not Setting Resource Limits

When creating pods in Kubernetes, if you do not set resource requests and limits, it can lead to several issues:

1. **Overconsumption of Resources**: Without limits, a single pod can consume all available CPU or memory on a node. This can starve other pods of the necessary resources, leading to degraded performance or even crashes.

2. **Resource Contention**: In a shared environment, multiple pods might compete for the same resources. Without limits, pods with higher demands might take more than their fair share, causing contention and instability in the cluster.

3. **Node Overload**: If multiple pods on the same node do not have limits, the node can become overloaded. This can lead to out-of-memory (OOM) errors, CPU throttling, and other issues that might bring down the node.

4. **Unpredictable Behavior**: Without defined requests, Kubernetes cannot properly schedule pods. This might lead to unpredictable performance, as the scheduler does not know the minimum resources required for the pod to run.

### 5. Comparing CPU and Memory

#### Key Differences
- **Compressibility**: CPU is compressible; Memory is not.
- **Allocation**: CPU can be adjusted dynamically; Memory allocation is fixed once assigned.
- **Units**: CPU is measured in millicores (m); Memory is measured in MB, MiB, GB, GiB, etc.

### 6. Practical Example

#### Creating a Pod with Defined Resources
Let’s create a pod with specific CPU and memory requests and limits.

**YAML File Example:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
  namespace: example-ns
spec:
  containers:
  - name: nginx-container
    image: nginx
    resources:
      requests:
        memory: "256Mi"
        cpu: "100m"
      limits:
        memory: "512Mi"
        cpu: "500m"
```

- **Requests**: This is the minimum amount of resources that Kubernetes guarantees to the pod. If the node cannot provide the requested amount of CPU or memory, the pod will not be scheduled.
  
- **Limits**: This defines the maximum amount of resources the pod can use. If the pod tries to exceed the defined limits, it will be throttled (for CPU) or terminated (for memory).

In this example:
- The pod requests **256Mi** of memory and **100m** of CPU, meaning Kubernetes will guarantee these resources.
- The pod can use up to **512Mi** of memory and **500m** of CPU, but no more. If the pod attempts to use more than these limits, it will be restricted.

#### Viewing Resource Allocation
To check how much CPU and memory are allocated to a pod:

```bash
kubectl describe pod example-pod --namespace=example-ns
```

You’ll see detailed information about CPU and memory allocation under the `Resources` section.

### 7. Conclusion
Understanding the differences between CPU and memory in Kubernetes and the importance of setting proper resource requests and limits is crucial for effectively managing resources in your cluster. By properly defining CPU and memory requests and limits, you can ensure that your applications run efficiently without overloading your nodes or affecting other pods.
