## Managing Resource Quotas and LimitRanges in Kubernetes

---

### Table of Contents

1. [Introduction to Resource Quotas and LimitRanges](#introduction-to-resource-quotas-and-limitranges)
2. [How Pods Interact with Resource Quotas and LimitRanges](#how-pods-interact-with-resource-quotas-and-limitaranges)
   - [How Resource Quotas Affect Pods](#how-resource-quotas-affect-pods)
   - [Can Pods Directly Have Resource Quotas?](#can-pods-directly-have-resource-quotas)
   - [How Pods Adhere to LimitRanges](#how-pods-adhere-to-limitaranges)
   - [What Happens if Pod Requests Exceed LimitRanges?](#what-happens-if-pod-requests-exceed-limitaranges)
   - [What Happens if Pod's Requests Are Lower than DefaultRequest?](#what-happens-if-pods-requests-are-lower-than-defaultrequest)
   - [Handling Multiple LimitRanges](#handling-multiple-limitaranges)
3. [Understanding `limits`, `default`, and `defaultRequest`](#understanding-limits-default-and-defaultrequest)
4. [Comprehensive Coverage of Resource Quotas](#comprehensive-coverage-of-resource-quotas)
   - [How Resource Quotas Work](#how-resource-quotas-work)
   - [Resource Quota Scenarios](#resource-quota-scenarios)
   - [Resource Quota YAML Examples](#resource-quota-yaml-examples)
   - [Resource Quota Validation and Outputs](#resource-quota-validation-and-outputs)
5. [Comprehensive Coverage of LimitRanges](#comprehensive-coverage-of-limitaranges)
   - [LimitRange Scenarios](#limitrange-scenarios)
   - [LimitRange YAML Examples](#limitrange-yaml-examples)
6. [Example: Resource Quotas and LimitRanges Together](#example-resource-quotas-and-limitranges-together)
7. [Conclusion](#conclusion)

---

### Introduction to Resource Quotas and LimitRanges

**Resource Quotas** and **LimitRanges** are key Kubernetes features that allow for efficient resource management within a cluster. They ensure that resources (CPU, memory, storage, etc.) are fairly allocated across different applications or teams using the same cluster. While **Resource Quotas** are applied at the **namespace level** to limit the total resource consumption in that namespace, **LimitRanges** are used to control the resource usage at the **Pod or container level**.

[Back to TOC](#table-of-contents)

---

### How Pods Interact with Resource Quotas and LimitRanges

#### How Resource Quotas Affect Pods

**Resource Quotas** limit the total resource consumption for all Pods in a namespace. When a Pod is created, Kubernetes checks the combined resource usage of all Pods in the namespace against the **Resource Quota**. If the resource consumption exceeds the quota, the Pod will be rejected.

For example, if the Resource Quota for CPU in a namespace is set to 2 cores and the Pods already consume 1.5 cores, a new Pod cannot request more than 0.5 cores.

[Back to TOC](#table-of-contents)

---

#### Can Pods Directly Have Resource Quotas?

No, Pods cannot directly have **Resource Quotas**. Resource Quotas are enforced at the namespace level. However, Pods can define **resource requests** and **limits**, and these values will be checked against both the **LimitRanges** and the **Resource Quotas** in the namespace. 

[Back to TOC](#table-of-contents)

---

#### How Pods Adhere to LimitRanges

A **LimitRange** applies specific resource limits (e.g., CPU, memory) to each Pod or container within a namespace. If a Pod does not specify its own resource requests or limits, Kubernetes automatically applies the default values from the **LimitRange**. 

If a Pod’s resource requests exceed the **LimitRange**, Kubernetes blocks the creation of the Pod.

[Back to TOC](#table-of-contents)

---

#### What Happens if Pod Requests Exceed LimitRanges?

If a Pod requests resources that exceed the **LimitRange**, Kubernetes will return an error message and reject the Pod.

Example Error:
```plaintext
Error from server (Forbidden): pods "example-pod" is forbidden: 
maximum cpu usage per Container is 500m, but request is 600m
```

[Back to TOC](#table-of-contents)

---

#### What Happens if Pod's Requests Are Lower than DefaultRequest?

If a Pod requests fewer resources than the **defaultRequest** in the **LimitRange**, Kubernetes will adjust the request to meet the default minimum values. 

For example, if the **LimitRange** specifies a **defaultRequest** of 250m CPU but the Pod requests 100m CPU, Kubernetes will override the request and set it to 250m CPU.

[Back to TOC](#table-of-contents)

---

#### Handling Multiple LimitRanges

Kubernetes allows multiple **LimitRanges** in the same namespace. When there are multiple **LimitRanges**, Kubernetes applies them independently. The key rules are:
- **Non-conflicting LimitRanges**: Each LimitRange can control different resources (e.g., one for CPU, one for memory). Kubernetes applies both sets of constraints without issue.
- **Conflicting LimitRanges**: If multiple LimitRanges set conflicting values for the same resource, Kubernetes ensures that Pods must meet the constraints of all applicable LimitRanges. The Pod will be rejected if it violates any of the constraints.

Example with two LimitRanges:

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

In this case:
- **cpu-limitrange** applies only to CPU limits.
- **memory-limitrange** applies only to memory limits.

Kubernetes applies both LimitRanges independently.

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

**Resource Quotas** operate at the **namespace level**. They limit the total amount of CPU, memory, storage, and other resources that all the Pods in a namespace can collectively consume. 

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

1. **Basic CPU and Memory Resource Quota**:

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
    requests.storage: "100Gi"

  # Total storage requested by all PVCs cannot exceed 100GiB
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

This output shows the current resource usage and the limits enforced by the **Resource Quota**. If any limit is reached, new Pods or other objects will be blocked.

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

---

### Example: Resource Quotas and LimitRanges Together

In this example, both **Resource Quotas** and **LimitRanges** are applied together in a namespace.

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
- **Resource Quota**: The total CPU and memory requests/limits across the entire namespace cannot exceed the defined values.
- **LimitRange**: Defines the default and maximum values for individual Pods and containers.
- **Pod**: This Pod explicitly defines its resource requests and limits, which must conform to the **LimitRange** values and not exceed the **Resource Quota** for the namespace.

[Back to TOC](#table-of-contents)

---

### Conclusion

By combining **Resource Quotas** and **LimitRanges**, Kubernetes ensures resources are allocated efficiently at both the **namespace** and **Pod** levels. While **Resource Quotas** control overall namespace resource consumption, **LimitRanges** apply constraints on individual Pods or containers. Together, these features help maintain a balanced and well-managed cluster environment.

[Back to TOC](#table-of-contents)

