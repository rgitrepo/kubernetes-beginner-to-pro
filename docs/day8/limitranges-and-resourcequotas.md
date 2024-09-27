## Managing Resource Quotas and LimitRanges

---

### Table of Contents

1. [Introduction to Resource Quotas and LimitRanges](#introduction-to-resource-quotas-and-limitranges)
2. [How Pods Interact with Resource Quotas and LimitRanges](#how-pods-interact-with-resource-quotas-and-limitaranges)
   - [How Resource Quotas Affect Pods](#how-resource-quotas-affect-pods)
   - [Can Pods Directly Have Resource Quotas?](#can-pods-directly-have-resource-quotas)
   - [How Pods Adhere to LimitRanges](#how-pods-adhere-to-limitaranges)
   - [What Happens if Pod Requests Exceed LimitRanges?](#what-happens-if-pod-requests-exceed-limitaranges)
   - [What Happens if Pod's Requests Are Lower than DefaultRequest?](#what-happens-if-pods-requests-are-lower-than-defaultrequest)
   - [Handling Multiple LimitRanges](#handling-multiple-limitranges)
   - [Enforcing Multiple Resource Limits](#enforcing-multiple-resource-limits)
3. [Understanding `limits`, `default`, and `defaultRequest`](#understanding-limits-default-and-defaultrequest)
4. [Comprehensive Coverage of Resource Quotas](#comprehensive-coverage-of-resource-quotas)
   - [How Resource Quotas Work](#how-resource-quotas-work)
   - [Resource Quota Scenarios](#resource-quota-scenarios)
   - [Resource Quota YAML Examples](#resource-quota-yaml-examples)
   - [Resource Quota Validation and Outputs](#resource-quota-validation-and-outputs)
5. [Comprehensive Coverage of LimitRanges](#comprehensive-coverage-of-limitaranges)
   - [LimitRange Scenarios](#limitrange-scenarios)
   - [LimitRange YAML Examples](#limitrange-yaml-examples)
6. [Example: Resource Quotas and LimitRanges Together](#example-resource-quotas-and-limitaranges-together)
7. [Resolving Conflicts with Multiple LimitRanges](#resolving-conflicts-with-multiple-limitaranges)
8. [Conclusion](#conclusion)

---

### Introduction to Resource Quotas and LimitRanges

**Resource Quotas** and **LimitRanges** are essential tools in Kubernetes for managing resource consumption at both the **namespace** and **Pod** levels. These tools help enforce fair resource allocation, prevent resource exhaustion, and optimize resource utilization in a multi-tenant environment.

- **Resource Quotas**: Applied at the namespace level, limiting the total resource consumption for all Pods and resources in that namespace.
- **LimitRanges**: Applied at the Pod/container level, controlling the individual resource usage within a namespace.

[Back to TOC](#table-of-contents)

---

### How Pods Interact with Resource Quotas and LimitRanges

#### How Resource Quotas Affect Pods

**Resource Quotas** limit the total resource consumption for all Pods in a namespace. When a Pod is created, Kubernetes checks the combined resource usage of all Pods in the namespace against the **Resource Quota**. If the resource consumption exceeds the quota, the Pod will be rejected.

For example, if the Resource Quota for CPU in a namespace is set to 2 cores and the Pods already consume 1.5 cores, a new Pod cannot request more than 0.5 cores.

[Back to TOC](#table-of-contents)

---

#### Can Pods Directly Have Resource Quotas?

No, Pods cannot directly have **Resource Quotas**. Resource Quotas are enforced at the namespace level, affecting all resources within that namespace. However, Pods can define **resource requests** and **limits**, which are checked against both the **LimitRanges** and the **Resource Quotas** in the namespace.

[Back to TOC](#table-of-contents)

---

#### How Pods Adhere to LimitRanges

In Kubernetes, a **Pod** adheres to a **LimitRange** based on the **namespace** it belongs to. Here's how this interaction works:

1. **Namespace Context**: When a **Pod** is created, Kubernetes checks which **namespace** the Pod belongs to. If a **LimitRange** is defined in that namespace, Kubernetes enforces the limits on the Pod.
   
2. **Default Resource Requests/Limit Inheritance**: If a Pod does not specify resource requests or limits, Kubernetes automatically applies the default values from the **LimitRange**.

3. **Validation on Pod Creation**: If a Pod specifies its own resource requests or limits, Kubernetes checks these values against the **LimitRange**. If the requests exceed the **LimitRange** values, the Pod will be rejected, and an error will be returned.

4. **No Need for Explicit Mapping**: A Pod doesn’t need to directly reference a **LimitRange**. Kubernetes enforces the constraints automatically for all Pods in the namespace.

[Back to TOC](#table-of-contents)

---

#### What Happens if Pod Requests Exceed LimitRanges?

If a Pod requests resources that exceed the **LimitRange**, Kubernetes returns an error message and rejects the Pod.

Example error:
```plaintext
Error from server (Forbidden): pods "example-pod" is forbidden: 
maximum cpu usage per Container is 500m, but request is 600m
```

[Back to TOC](#table-of-contents)

---

#### What Happens if Pod's Requests Are Lower than DefaultRequest?

If a Pod's resource requests are lower than the **defaultRequest** in the **LimitRange**, Kubernetes will adjust the request to meet the minimum values specified by the LimitRange. For instance, if the **defaultRequest** specifies 250m CPU and the Pod requests 100m, Kubernetes will automatically set the request to 250m.

[Back to TOC](#table-of-contents)

---

#### Handling Multiple LimitRanges

When multiple **LimitRanges** exist within a namespace, Kubernetes applies each LimitRange independently:

1. **Non-conflicting Constraints**: If the **LimitRanges** specify limits for different resources (e.g., CPU and memory), Kubernetes enforces all constraints without issue.
   
2. **Conflicting Constraints**: If multiple **LimitRanges** define conflicting values for the same resource, Kubernetes requires Pods to meet all applicable constraints. If a Pod violates any of the constraints, it will be rejected.

Example of handling multiple LimitRanges:

```yaml
# LimitRange for CPU
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limitrange
  namespace: example-namespace
spec:
  limits:
  - default:
      cpu: "500m"
    defaultRequest:
      cpu: "250m"
    type: Container

# LimitRange for Memory
apiVersion: v1
kind: LimitRange
metadata:
  name: memory-limitrange
  namespace: example-namespace
spec:
  limits:
  - default:
      memory: "512Mi"
    defaultRequest:
      memory: "256Mi"
    type: Container
```

In this scenario, Kubernetes enforces the CPU limits from **cpu-limitrange** and the memory limits from **memory-limitrange** independently.

[Back to TOC](#table-of-contents)

---

#### Enforcing Multiple Resource Limits

Kubernetes ensures that Pods comply with multiple resource limits simultaneously. When there are constraints on both CPU and memory, the Pod must adhere to both limits.

For example:
1. A **LimitRange** for CPU specifies a default limit of 500m CPU, and another **LimitRange** for memory specifies a default limit of 512Mi memory.
2. A Pod must conform to both the CPU and memory constraints. If either limit is exceeded, the Pod will not be created.

This ensures that Pods cannot bypass constraints by adhering to only one resource limit.

[Back to TOC](#table-of-contents)

---

### Understanding `limits`, `default`, and `defaultRequest`

#### Limits:
- Defines the **maximum** resources that a container or Pod can use.
- Kubernetes enforces these values to prevent over-consumption of resources.

#### Default:
- The **default limit** applied if a Pod does not specify its own limits.
- If no explicit resource limit is set, Kubernetes uses this value to cap the Pod’s resource consumption.

#### DefaultRequest:
- Specifies the **minimum resources** that a container or Pod will request.
- Kubernetes uses this value to schedule Pods, ensuring that a Pod gets the minimum resources it needs to function properly.

[Back to TOC](#table-of-contents)

---

### Comprehensive Coverage of Resource Quotas

#### How Resource Quotas Work

**Resource Quotas** operate at the **namespace level** and control the total amount of CPU, memory, storage, and other resources that all the Pods in a namespace can collectively consume.

When a new Pod is created, Kubernetes checks if the new Pod's resource requests, when added to the total current usage, will exceed the Resource Quota. If so, the Pod creation is denied.

[Back to TOC](#table-of-contents)

---

#### Resource Quota Scenarios

1. **Basic CPU and Memory Quota**:
   - Limit the total CPU and memory resources requested by all Pods in a namespace.

2. **Limiting Object Counts**:
   - Control the number of objects (Pods, Services, PersistentVolumeClaims, etc.) that can be created in the namespace.

3. **Storage Quotas**:
   - Restrict the amount of storage that PersistentVolumeClaims can use.

4. **Combined Quota Types**:
   - Limit CPU, memory, object counts, and storage within a single quota.

[Back to TOC](#table-of-contents)

---

#### Resource Quota YAML Examples

1

. **Basic CPU and Memory Resource Quota**:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: example-namespace
spec:
  hard:
    requests.cpu: "2"         # Total CPU requests across all Pods cannot exceed 2 cores
    requests.memory: "4Gi"    # Total memory requests cannot exceed 4GiB
    limits.cpu: "4"           # Total CPU usage cannot exceed 4 cores
    limits.memory: "8Gi"      # Total memory usage cannot exceed 8GiB
```

2. **Resource Quota with Object Count**:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: object-count-quota
  namespace: example-namespace
spec:
  hard:
    pods: "10"                # No more than 10 Pods can be created in this namespace
    services: "5"             # No more than 5 Services can be created
    persistentvolumeclaims: "5" # No more than 5 PersistentVolumeClaims (PVCs) can be created
```

3. **Storage Resource Quota**:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: storage-quota
  namespace: example-namespace
spec:
  hard:
    requests.storage: "100Gi"  # Total storage requested by all PVCs cannot exceed 100GiB
    persistentvolumeclaims: "10" # No more than 10 PVCs can be created
```

[Back to TOC](#table-of-contents)

---

#### Resource Quota Validation and Outputs

Once the Resource Quota is applied using `kubectl apply`, you can check the current usage and available quota:

```bash
kubectl get resourcequotas -n example-namespace
```

To see more details:

```bash
kubectl describe resourcequota compute-quota -n example-namespace
```

**Example Output**:

```plaintext
Name:            compute-quota
Namespace:       example-namespace
Resource         Used   Hard
--------         ----   ----
requests.cpu     1.5    2
requests.memory  3Gi    4Gi
limits.cpu       2      4
limits.memory    4Gi    8Gi
```

[Back to TOC](#table-of-contents)

---

### Comprehensive Coverage of LimitRanges

#### LimitRange Scenarios

1. **Basic CPU and Memory Limits**:
   - Enforce the maximum and minimum resource usage for individual Pods or containers.

2. **Multiple LimitRanges**:
   - Different LimitRanges can control different resource types (e.g., one for CPU and another for memory).

3. **Default Resource Requests and Limits**:
   - Apply default values to Pods that do not explicitly define their own requests or limits.

[Back to TOC](#table-of-contents)

---

#### LimitRange YAML Examples

1. **Basic CPU and Memory LimitRange**:

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: basic-limits
  namespace: example-namespace
spec:
  limits:
  - default:
      cpu: "500m"           # Default maximum CPU limit for containers
      memory: "512Mi"       # Default maximum memory limit for containers
    defaultRequest:
      cpu: "250m"           # Default minimum CPU request (scheduler uses this)
      memory: "256Mi"       # Default minimum memory request (scheduler uses this)
    type: Container         # These limits and requests apply to containers in Pods
```

2. **Multiple LimitRanges (CPU and Memory)**:

```yaml
# LimitRange for CPU
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limitrange
  namespace: example-namespace
spec:
  limits:
  - default:
      cpu: "500m"
    defaultRequest:
      cpu: "250m"
    type: Container

# LimitRange for Memory
apiVersion: v1
kind: LimitRange
metadata:
  name: memory-limitrange
  namespace: example-namespace
spec:
  limits:
  - default:
      memory: "512Mi"
    defaultRequest:
      memory: "256Mi"
    type: Container
```

In this case, Kubernetes enforces both the CPU and memory constraints independently. Pods must adhere to both limits.

[Back to TOC](#table-of-contents)

---

### Example: Resource Quotas and LimitRanges Together

#### Resource Quota YAML:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: combined-quota
  namespace: example-namespace
spec:
  hard:
    requests.cpu: "3"           # Total CPU requests cannot exceed 3 cores
    requests.memory: "6Gi"      # Total memory requests cannot exceed 6GiB
    limits.cpu: "6"             # Total CPU usage cannot exceed 6 cores
    limits.memory: "12Gi"       # Total memory usage cannot exceed 12GiB
```

#### LimitRange YAML:

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: combined-limitrange
  namespace: example-namespace
spec:
  limits:
  - default:
      cpu: "500m"              # Default CPU limit per container
      memory: "512Mi"          # Default memory limit per container
    defaultRequest:
      cpu: "250m"              # Default minimum CPU request per container
      memory: "256Mi"          # Default minimum memory request per container
    type: Container
```

#### Pod YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: combined-pod
  namespace: example-namespace
spec:
  containers:
  - name: nginx-container
    image: nginx
    resources:
      requests:
        cpu: "300m"            # Pod explicitly requests 300m CPU
        memory: "300Mi"        # Pod explicitly requests 300Mi memory
      limits:
        cpu: "600m"            # Pod has a limit of 600m CPU
        memory: "600Mi"        # Pod has a memory limit of 600Mi
```

**Explanation**:
- **Resource Quota**: Limits the total CPU and memory usage across the entire namespace.
- **LimitRange**: Defines default and maximum values for each individual Pod or container.
- **Pod**: Specifies explicit resource requests and limits. These values must conform to the **LimitRange** constraints, and the total consumption across the namespace must not exceed the **Resource Quota**.

[Back to TOC](#table-of-contents)

---

### Resolving Conflicts with Multiple LimitRanges

When there are **multiple LimitRanges** for the **same resource type** (e.g., CPU or memory) with different values, Kubernetes handles these conflicts by enforcing **all applicable LimitRanges**. Pods must adhere to the most restrictive values across all LimitRanges.

#### Example: Conflicting LimitRanges for CPU and Memory

Let’s say we have two different LimitRanges for both CPU and memory in the same namespace with different values.

##### LimitRange 1 (cpu-limits):
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limits
  namespace: example-namespace
spec:
  limits:
  - default:
      cpu: "600m"         # Default maximum CPU limit
    defaultRequest:
      cpu: "300m"         # Default minimum CPU request
    type: Container
```

##### LimitRange 2 (memory-limits):
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: memory-limits
  namespace: example-namespace
spec:
  limits:
  - default:
      memory: "700Mi"     # Default maximum memory limit
    defaultRequest:
      memory: "400Mi"     # Default minimum memory request
    type: Container
```

In this scenario:
- **cpu-limits** applies a default maximum CPU limit of 600m and a default minimum CPU request of 300m.
- **memory-limits** applies a default maximum memory limit of 700Mi and a default minimum memory request of 400Mi.

#### Pod Definition:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: conflicting-pod
  namespace: example-namespace
spec:
  containers:
  - name: nginx-container
    image: nginx
    resources:
      requests:
        cpu: "400m"        # Pod requests 400m CPU
        memory: "450Mi"    # Pod requests 450Mi memory
      limits:
        cpu: "550m"        # Pod defines a limit of 550m CPU
        memory: "650Mi"    # Pod defines a memory limit of 650Mi
```

#### Resolution:
- The Pod requests **400m CPU** and sets a CPU limit of **550m**.
- The Pod requests **450Mi memory** and sets a memory limit of **650Mi**.
- **CPU**: The Pod satisfies the CPU constraints, as 400m is within the range of 300m (minimum) to 600m (maximum).
- **Memory**: The Pod satisfies the memory constraints, as 450Mi is within the range of 400Mi (minimum) to 700Mi (maximum).

Kubernetes will allow the Pod to be created because it satisfies both LimitRanges for CPU and memory.

If the Pod had exceeded any of the limits (e.g., requesting 700m CPU or 750Mi memory), Kubernetes would reject the Pod with an error message.

[Back

 to TOC](#table-of-contents)

---

### Conclusion

By combining **Resource Quotas** and **LimitRanges**, Kubernetes ensures that resources are efficiently managed at both the **namespace** and **Pod** levels. Pods must adhere to both the constraints set by the **LimitRange** and the overall resource limits defined in the **Resource Quota**. Additionally, when there are multiple **LimitRanges** for the same resource, Kubernetes ensures that Pods comply with all constraints, rejecting any Pod that violates even one of the limits.

[Back to TOC](#table-of-contents)

