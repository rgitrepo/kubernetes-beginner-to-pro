
### Node Affinity and Pod Affinity/Anti-Affinity in Kubernetes

---

#### Table of Contents

1. [Introduction](#introduction)
2. [Understanding Affinity and Anti-Affinity](#understanding-affinity-and-anti-affinity)
   - [What is Node Affinity?](#what-is-node-affinity)
   - [What is Pod Affinity and Anti-Affinity?](#what-is-pod-affinity-and-anti-affinity)
3. [Node Affinity](#node-affinity)
   - [Required vs. Preferred Node Affinity](#required-vs-preferred-node-affinity)
   - [Scheduling Modes](#scheduling-modes-table)
   - [YAML Example of Node Affinity](#yaml-example-of-node-affinity)
4. [Pod Affinity and Anti-Affinity](#pod-affinity-and-anti-affinity)
   - [Required vs. Preferred Pod Affinity/Anti-Affinity](#required-vs-preferred-pod-affinityanti-affinity)
   - [YAML Example of Pod Affinity and Anti-Affinity](#yaml-example-of-pod-affinity-and-anti-affinity)
5. [Operators in Affinity Rules](#operators-in-affinity-rules)
   - [Operators Table](#operators-table)
   - [Examples of `matchExpressions`](#examples-of-matchexpressions)
6. [Practical Examples](#practical-examples)
   - [Applying Node Affinity](#applying-node-affinity)
   - [Applying Pod Affinity/Anti-Affinity](#applying-pod-affinityanti-affinity)
   - [Verifying Pod Scheduling](#verifying-pod-scheduling)
7. [Conclusion](#conclusion)

---

### Introduction

Affinity and Anti-Affinity rules provide more granular control over Pod scheduling in Kubernetes compared to simple node labels. With these rules, you can define where Pods should or should not be placed based on node or Pod labels. This tutorial will cover Node Affinity, Pod Affinity, and Pod Anti-Affinity, along with their scheduling modes, operators, and YAML examples.

[Back to TOC](#table-of-contents)

---

### Understanding Affinity and Anti-Affinity

#### What is Node Affinity?

Node Affinity is used to constrain which nodes a Pod can be scheduled on, based on node labels. It offers more flexibility than `nodeSelector`, with both hard and soft rules for scheduling.

#### What is Pod Affinity and Anti-Affinity?

Pod Affinity and Anti-Affinity allow you to control Pod scheduling based on the labels of other Pods running on the same or different nodes.

- **Pod Affinity**: Ensures that Pods are scheduled on nodes near other Pods with specific labels.
- **Pod Anti-Affinity**: Ensures that Pods are scheduled away from nodes with other Pods that match specific labels.

[Back to TOC](#table-of-contents)

---

### Node Affinity

#### Required vs. Preferred Node Affinity

- **Required Node Affinity**: (`requiredDuringSchedulingIgnoredDuringExecution`) is a hard constraint. If no nodes match the specified criteria, the Pod will not be scheduled.
- **Preferred Node Affinity**: (`preferredDuringSchedulingIgnoredDuringExecution`) is a soft constraint. If no nodes match the specified criteria, the Pod will still be scheduled on other available nodes.

#### Scheduling Modes Table

| Mode | Description | Behavior |
|---|---|---|
| `requiredDuringSchedulingIgnoredDuringExecution` | Node must satisfy the condition when scheduling but ignores changes during execution. | Hard constraint, Pod won’t be scheduled if no matching nodes are found. |
| `preferredDuringSchedulingIgnoredDuringExecution` | Scheduler prefers nodes that satisfy the condition but can still schedule on others if necessary. | Soft constraint, Pod will be scheduled even if no matching nodes are found. |
| `requiredduringschedulingrequiredduringexecution` (Planned) | Node must satisfy condition during both scheduling and execution. | Not currently supported. Expected to be a stricter version of `requiredDuringSchedulingIgnoredDuringExecution`. |
| `preferredduringschedulingrequiredduringexecution` (Planned) | Node is preferred during both scheduling and execution. | Expected to enforce soft constraints during both scheduling and execution. |

[Back to TOC](#table-of-contents)

---

#### YAML Example of Node Affinity

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
  - name: nginx
    image: nginx
```

In this example:
- **Required**: Pod will only be scheduled on nodes in the `antarctica-east1` or `antarctica-west1` zones.
- **Preferred**: Pod prefers nodes with the label `another-node-label-key=another-node-label-value`.

[Back to TOC](#table-of-contents)

---

### Pod Affinity and Anti-Affinity

#### Required vs. Preferred Pod Affinity/Anti-Affinity

- **Required Pod Affinity/Anti-Affinity**: (`requiredDuringSchedulingIgnoredDuringExecution`) ensures Pods are scheduled only if criteria are met.
- **Preferred Pod Affinity/Anti-Affinity**: (`preferredDuringSchedulingIgnoredDuringExecution`) tries to respect criteria but will schedule Pods even if none meet the conditions.

#### YAML Example of Pod Affinity and Anti-Affinity

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
  - name: nginx
    image: nginx
```

- **Pod Affinity**: The Pod will only be scheduled in zones that already have a Pod with the label `security=S1`.
- **Pod Anti-Affinity**: The Pod prefers to avoid zones that already have Pods with the label `security=S2`.

[Back to TOC](#table-of-contents)

---

### Operators in Affinity Rules

Affinity rules use specific operators to match expressions against node or Pod labels.

#### Operators Table

| Operator | Description | Example |
|---|---|---|
| `In` | Matches when the label value is in a list of specified values. | `topology.kubernetes.io/zone In (antarctica-east1, antarctica-west1)` |
| `NotIn` | Matches when the label value is not in a list of specified values. | `security NotIn (S2, S3)` |
| `Exists` | Matches when the label key is present, regardless of value. | `security Exists` |
| `DoesNotExist` | Matches when the label key is not present. | `security DoesNotExist` |
| `Gt` | Matches when the label value is greater than a specified value. | `version Gt 2` |
| `Lt` | Matches when the label value is less than a specified value. | `version Lt 5` |

#### Examples of `matchExpressions`

```yaml
matchExpressions:
- key: topology.kubernetes.io/zone
  operator: In
  values:
  - antarctica-east1
  - antarctica-west1
- key: security
  operator: NotIn
  values:
  - S2
  - S3
```

In this example:
- The first expression matches nodes in the specified zones.
- The second expression avoids nodes with the `security` label set to `S2` or `S3`.

[Back to TOC](#table-of-contents)

---

### Practical Examples

#### Applying Node Affinity

Save the node affinity YAML file as `pod-with-node-affinity.yaml` and apply it:

```bash
kubectl apply -f pod-with-node-affinity.yaml
```

#### Applying Pod Affinity/Anti-Affinity

Save the pod affinity/anti-affinity YAML file as `pod-with-pod-affinity.yaml` and apply it:

```bash
kubectl apply -f pod-with-pod-affinity.yaml
```

#### Verifying Pod Scheduling

To verify where the Pods have been scheduled:

```bash
kubectl get pods -o wide
```

This command will show the nodes on which Pods were scheduled, allowing you to confirm that affinity/anti-affinity rules were respected.

[Back to TOC](#table-of-contents)

---

### Conclusion

By leveraging Node Affinity and Pod Affinity/Anti-Affinity in Kubernetes, you gain control over where Pods are scheduled, ensuring your cluster runs optimally. This tutorial covered both affinity types, scheduling

 modes, operators, and practical examples, equipping you to implement these features effectively in your Kubernetes environment.

[Back to TOC](#table-of-contents) 
