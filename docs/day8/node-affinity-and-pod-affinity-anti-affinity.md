### Node Affinity and Pod Affinity/Anti-Affinity in Kubernetes

#### Table of Contents

1. [Introduction](#introduction)
2. [Understanding Affinity and Anti-Affinity](#understanding-affinity-and-anti-affinity)
   - [What is Node Affinity?](#what-is-node-affinity)
   - [What is Pod Affinity and Anti-Affinity?](#what-is-pod-affinity-and-anti-affinity)
3. [Node Affinity](#node-affinity)
   - [Required vs. Preferred Node Affinity](#required-vs-preferred-node-affinity)
   - [YAML Example of Node Affinity](#yaml-example-of-node-affinity)
4. [Pod Affinity and Anti-Affinity](#pod-affinity-and-anti-affinity)
   - [Required vs. Preferred Pod Affinity/Anti-Affinity](#required-vs-preferred-pod-affinityanti-affinity)
   - [YAML Example of Pod Affinity and Anti-Affinity](#yaml-example-of-pod-affinity-and-anti-affinity)
5. [Practical Examples](#practical-examples)
   - [Applying Node Affinity](#applying-node-affinity)
   - [Applying Pod Affinity/Anti-Affinity](#applying-pod-affinityanti-affinity)
   - [Verifying Pod Scheduling](#verifying-pod-scheduling)
6. [Conclusion](#conclusion)

---

### Introduction

In Kubernetes, affinity and anti-affinity provide more granular control over where Pods are scheduled compared to basic `nodeSelector`. By defining rules based on node labels or other Pods' labels, you can ensure that your Pods are placed in the most optimal locations within your cluster. This tutorial will guide you through node affinity, pod affinity, and pod anti-affinity, with YAML examples and practical steps to apply these concepts.

[Back to TOC](#table-of-contents)

---

### Understanding Affinity and Anti-Affinity

#### What is Node Affinity?

Node Affinity is a way to constrain which nodes a Pod can be scheduled on based on node labels. It offers more flexibility and expressiveness than `nodeSelector` by allowing you to define both hard (required) and soft (preferred) rules for scheduling.

#### What is Pod Affinity and Anti-Affinity?

Pod Affinity and Anti-Affinity allow you to constrain which nodes your Pods can be scheduled on based on the labels of Pods already running on those nodes. This enables you to co-locate or spread out Pods according to specific rules.

- **Pod Affinity**: Ensures that a Pod is scheduled near other Pods that match certain labels.
- **Pod Anti-Affinity**: Ensures that a Pod is scheduled away from other Pods that match certain labels.

[Back to TOC](#table-of-contents)

---

### Node Affinity

#### Required vs. Preferred Node Affinity

- **Required Node Affinity** (`requiredDuringSchedulingIgnoredDuringExecution`): The scheduler must find a node that meets the specified criteria, or the Pod will not be scheduled.
- **Preferred Node Affinity** (`preferredDuringSchedulingIgnoredDuringExecution`): The scheduler tries to find a node that meets the criteria, but if none are available, it will schedule the Pod on a node that doesn't match.

#### YAML Example of Node Affinity

Here's an example YAML file showing both required and preferred node affinity:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: topology.kubernetes.io/zone
            operator: In
            values:
            - antarctica-east1
            - antarctica-west1
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
  containers:
  - name: container
    image: nginx
```

- **Required**: The node must have a label `topology.kubernetes.io/zone` with the value `antarctica-east1` or `antarctica-west1`.
- **Preferred**: The node preferably has a label `another-node-label-key` with the value `another-node-label-value`.

[Back to TOC](#table-of-contents)

---

### Pod Affinity and Anti-Affinity

#### Required vs. Preferred Pod Affinity/Anti-Affinity

- **Required Pod Affinity/Anti-Affinity** (`requiredDuringSchedulingIgnoredDuringExecution`): The Pod will only be scheduled if the specified criteria are met. This is a "hard" constraint.
- **Preferred Pod Affinity/Anti-Affinity** (`preferredDuringSchedulingIgnoredDuringExecution`): The Pod will try to be scheduled according to the criteria, but if it can't, it will still be scheduled. This is a "soft" constraint.

#### YAML Example of Pod Affinity and Anti-Affinity

Here's an example YAML file showing both pod affinity and pod anti-affinity:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-pod-affinity
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: security
            operator: In
            values:
            - S1
        topologyKey: topology.kubernetes.io/zone
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: security
              operator: In
              values:
              - S2
          topologyKey: topology.kubernetes.io/zone
  containers:
  - name: container
    image: nginx
```

- **Pod Affinity**: The Pod will only be scheduled in a zone that already has a Pod with the label `security=S1`.
- **Pod Anti-Affinity**: The scheduler will try to avoid scheduling the Pod in a zone that has a Pod with the label `security=S2`.

[Back to TOC](#table-of-contents)

---

### Practical Examples

#### Applying Node Affinity

Save the node affinity YAML file as `pod-with-node-affinity.yaml` and apply it using the following command:

```bash
kubectl apply -f pod-with-node-affinity.yaml
```

This command will schedule the Pod according to the defined node affinity rules.

#### Applying Pod Affinity/Anti-Affinity

Save the pod affinity/anti-affinity YAML file as `pod-with-pod-affinity.yaml` and apply it using the following command:

```bash
kubectl apply -f pod-with-pod-affinity.yaml
```

This command will schedule the Pod based on the affinity and anti-affinity rules specified.

#### Verifying Pod Scheduling

To verify where the Pods have been scheduled, use the following command:

```bash
kubectl get pods -o wide
```

This will show the nodes on which the Pods have been scheduled, allowing you to confirm that the affinity/anti-affinity rules were respected.

[Back to TOC](#table-of-contents)

---

### Conclusion

Understanding and using affinity and anti-affinity in Kubernetes allows you to have fine-grained control over how and where your Pods are scheduled. By applying the concepts covered in this tutorial, including node affinity and pod affinity/anti-affinity, you can ensure that your workloads are optimally placed within your cluster.

This tutorial provided YAML examples and practical steps to help you implement these features effectively in your Kubernetes environment.

[Back to TOC](#table-of-contents)
