## Taints and Tolerations in Kubernetes

### Table of Contents
1. [Introduction](#introduction)
2. [Understanding Taints and Tolerations](#understanding-taints-and-tolerations)
   - [Taints](#taints)
   - [Tolerations](#tolerations)
3. [Managing Taints](#managing-taints)
   - [Managing Taints with Commands](#managing-taints-with-commands)
     - [Tainting a Node with a Command](#tainting-a-node-with-a-command)
     - [Removing a Taint with a Command](#removing-a-taint-with-a-command)
   - [Managing Taints with YAML Files](#managing-taints-with-yaml-files)
     - [Applying a Taint in a Node YAML File](#applying-a-taint-in-a-node-yaml-file)
4. [Managing Tolerations](#managing-tolerations)
   - [Managing Tolerations with Commands](#managing-tolerations-with-commands)
     - [Applying Tolerations Using Commands](#applying-tolerations-using-commands)
   - [Managing Tolerations with YAML Files](#managing-tolerations-with-yaml-files)
     - [Applying a Toleration in a Pod YAML File](#applying-a-toleration-in-a-pod-yaml-file)
     - [Removing a Toleration from a Pod](#removing-a-toleration-from-a-pod)
5. [Example Scenarios](#example-scenarios)
   - [Example 1: Tainting a Node to Prevent Scheduling Pods](#example-1-tainting-a-node-to-prevent-scheduling-pods)
   - [Example 2: Tainting a Node with `PreferNoSchedule`](#example-2-tainting-a-node-with-prefernoschedule)
   - [Example 3: Tainting a Node with `NoExecute`](#example-3-tainting-a-node-with-noexecute)
   - [Example 4: Applying Tolerations in Pod YAML](#example-4-applying-tolerations-in-pod-yaml)
6. [Master Node - Tainted by Default](#6-master-node---tainted-by-default)
7. [Summary](#summary)

---

### 1. Introduction

In Kubernetes, taints and tolerations work together to control which pods can be scheduled on specific nodes. Taints are applied to nodes and repel pods that do not have matching tolerations, while tolerations are applied to pods and allow them to be scheduled on tainted nodes. This tutorial will guide you through using taints and tolerations, both through commands and YAML configurations, and will include examples, outputs, and explanations to enhance your understanding.

[Back to TOC](#table-of-contents)

---

### 2. Understanding Taints and Tolerations

#### Taints

A taint is applied to a Kubernetes node and prevents pods from being scheduled on that node unless the pod has a corresponding **toleration**. Taints are key-value pairs with an associated effect that determines how the taint affects pod scheduling.

- **Key**: A label key applied to the node.
- **Value**: A label value paired with the key.
- **Effect**: Determines how the taint is enforced:
  - `NoSchedule`: Pods without matching tolerations will not be scheduled on the node. The existing one's if any without the toleration wont' be evicted.
  - `PreferNoSchedule`: The scheduler will try to avoid placing pods wihtout tolerations on the node, but it’s not enforced strictly.
  - `NoExecute`: Pods without matching tolerations will be evicted if they are running on the node. No new pods would be scheduled if they don't tolerate the taint.

[Back to TOC](#table-of-contents)

#### Tolerations

Tolerations allow pods to be scheduled on nodes with specific taints. A toleration in the pod’s specification must match the taint's key, value, and effect for the pod to be scheduled on the tainted node. Having a toleration **doesn't guarantee a pod will be scheduled on a node. It just has the permission to if it happens to be scheduled on that node.**

[Back to TOC](#table-of-contents)

---

### 3. Managing Taints

#### Managing Taints with Commands

##### Tainting a Node with a Command

To apply a taint to a node using a command:

```bash
kubectl taint nodes <node-name> <key>=<value>:<effect>
```

**Example:**

```bash
kubectl taint nodes node1 dedicated=ml-tasks:NoSchedule
```

**Output:**

```
node/node1 tainted
```

This command adds a taint to `node1` with the key `dedicated`, value `ml-tasks`, and effect `NoSchedule`. This means that only pods with a matching toleration can be scheduled on `node1`. Existing pods without toleration aren't evicted.

[Back to TOC](#table-of-contents)

##### Removing a Taint with a Command

To remove a taint from a node:

```bash
kubectl taint nodes <node-name> <key>=<value>:<effect>-
```

**Example:**

```bash
kubectl taint nodes node1 dedicated=ml-tasks:NoSchedule-
```

**Output:**

```
node/node1 untainted
```

This command removes the previously applied taint from `node1`.

[Back to TOC](#table-of-contents)

#### Managing Taints with YAML Files

##### Applying a Taint in a Node YAML File

While taints are typically applied using the `kubectl taint` command, you can also define taints in the node's YAML configuration if you are managing nodes declaratively.

**Example Node YAML:**

```yaml
apiVersion: v1
kind: Node
metadata:
  name: node1
spec:
  taints:
  - key: "dedicated"
    value: "ml-tasks"
    effect: "NoSchedule"
```
Note the values under taints are encoded in **double-quotes**.


**Applying the Configuration:**

```bash
kubectl apply -f node1.yaml
```

This YAML file taints the node `node1` with the `NoSchedule` effect.

[Back to TOC](#table-of-contents)

---

### 4. Managing Tolerations

#### Managing Tolerations with Commands

##### Applying Tolerations Using Commands

While tolerations are typically applied using YAML configurations, you can specify tolerations directly within pod specifications using the `kubectl run` command.

**Example:**

```bash
kubectl run my-pod --image=nginx --dry-run=client -o yaml > my-pod.yaml
```

Then, edit the `my-pod.yaml` file to include the toleration:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "ml-tasks"
    effect: "NoSchedule"
```

Apply the pod configuration:

```bash
kubectl apply -f my-pod.yaml
```

This toleration allows `my-pod` to be scheduled on a node tainted with `dedicated=ml-tasks:NoSchedule`.

[Back to TOC](#table-of-contents)

#### Managing Tolerations with YAML Files

##### Applying a Toleration in a Pod YAML File

Tolerations are typically defined within a pod's YAML specification to allow the pod to be scheduled on nodes with matching taints.

**Example Pod YAML:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "ml-tasks"
    effect: "NoSchedule"
```

**Applying the Configuration:**

```bash
kubectl apply -f my-pod.yaml
```

This YAML file configures the pod `my-pod` to tolerate the taint `dedicated=ml-tasks:NoSchedule`.

[Back to TOC](#table-of-contents)

##### Removing a Toleration from a Pod

To remove a toleration from a pod in Kubernetes, you must either delete and recreate the pod with the updated YAML file or modify the Deployment, StatefulSet, or ReplicaSet managing the pod.

###### 1. Using a Deployment, StatefulSet, or ReplicaSet

If your pod is managed by a Deployment, StatefulSet, or ReplicaSet:

1. **Edit the Deployment/StatefulSet/ReplicaSet YAML:**

   Find and edit the YAML file for your Deployment, StatefulSet, or ReplicaSet. Locate the `tolerations` section and remove the specific toleration.

   **Original YAML:**

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: my-deployment
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: my-app
     template:
       metadata:
         labels:
           app: my-app
       spec:
         containers:
         - name: my-container
           image: nginx
         tolerations:
         - key: "dedicated"
           operator: "Equal"
           value: "ml-tasks"
           effect: "NoSchedule"
   ```

   **Modified YAML (Toleration Removed):**

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: my-deployment
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: my-app
     template:
       metadata:
         labels:
           app: my-app
       spec:
         containers:


         - name: my-container
           image: nginx
   ```

2. **Apply the Updated YAML:**

   Apply the changes to the cluster using `kubectl apply`:

   ```bash
   kubectl apply -f my-deployment.yaml
   ```

   Kubernetes will update the pods managed by the Deployment, StatefulSet, or ReplicaSet, removing the toleration from the newly created pods.

###### 2. Directly on an Individual Pod

If you need to modify a standalone pod:

1. **Delete the Existing Pod:**

   First, delete the existing pod:

   ```bash
   kubectl delete pod <pod-name>
   ```

2. **Create a New Pod Without the Toleration:**

   Modify the pod's YAML to remove the toleration and then recreate the pod:

   **Original YAML:**

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: my-pod
   spec:
     containers:
     - name: my-container
       image: nginx
     tolerations:
     - key: "dedicated"
       operator: "Equal"
       value: "ml-tasks"
       effect: "NoSchedule"
   ```

   **Modified YAML (Toleration Removed):**

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: my-pod
   spec:
     containers:
     - name: my-container
       image: nginx
   ```

3. **Recreate the Pod:**

   Apply the updated YAML file to recreate the pod without the toleration:

   ```bash
   kubectl apply -f my-pod.yaml
   ```

[Back to TOC](#table-of-contents)

---

### 5. Example Scenarios

#### Example 1: Tainting a Node to Prevent Scheduling Pods

To prevent pods from being scheduled on `node1` unless they have a specific toleration:

```bash
kubectl taint nodes node1 dedicated=special-use:NoSchedule
```

**Toleration in Pod YAML:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: special-pod
spec:
  containers:
  - name: special-container
    image: nginx
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "special-use"
    effect: "NoSchedule"
```

This setup ensures that only pods with the `special-use` toleration are scheduled on `node1`.

[Back to TOC](#table-of-contents)

#### Example 2: Tainting a Node with `PreferNoSchedule`

To suggest that pods should avoid being scheduled on `node2`:

```bash
kubectl taint nodes node2 dedicated=low-priority:PreferNoSchedule
```

**Toleration in Pod YAML:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: low-priority-pod
spec:
  containers:
  - name: low-priority-container
    image: nginx
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "low-priority"
    effect: "PreferNoSchedule"
```

This setup indicates that `low-priority-pod` can be scheduled on `node2`, but the scheduler will prefer not to.

[Back to TOC](#table-of-contents)

#### Example 3: Tainting a Node with `NoExecute`

To immediately evict pods from `node3` if they don't have a matching toleration:

```bash
kubectl taint nodes node3 critical=true:NoExecute
```

**Toleration in Pod YAML:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: critical-pod
spec:
  containers:
  - name: critical-container
    image: nginx
  tolerations:
  - key: "critical"
    operator: "Equal"
    value: "true"
    effect: "NoExecute"
```

Pods without this toleration will be evicted from `node3` and new pods won't be scheduled.

[Back to TOC](#table-of-contents)

#### Example 4: Applying Tolerations in Pod YAML

For a pod to tolerate multiple taints:

**Pod YAML with Multiple Tolerations:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-tolerant-pod
spec:
  containers:
  - name: multi-container
    image: nginx
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "special-use"
    effect: "NoSchedule"
  - key: "critical"
    operator: "Equal"
    value: "true"
    effect: "NoExecute"
```

This pod can be scheduled on nodes tainted with both `dedicated=special-use:NoSchedule` and `critical=true:NoExecute`.

[Back to TOC](#table-of-contents)

---
### 6. Master Node - Tainted by Default
We've been talking about Worker Nodes mostly. But we also have a Master Node in a cluster, which is also like any other node and has all the capabilites of hosting a pod plus it runs all the management software. If you haven't noticed, the scheduler doesn't schedule any pods on the master node. Why is that? When the kubernetes cluster is first setup a taint is set on master node automatically that prevents any pods from being scheduled on this node. You can see this as well as modify this behiour if required. However, a best practice is to no deploy an application workload on the master node. 

**To see the tain on master node or controlplane**

```bash
kubectl describe node kubemaster | grep Taint
```


**Output:**
```bash
Taints: node-role.kubernetes.io/master:NoSchedule
```

**To remove the taint from master node or controlplane**

```bash
kubectl taint node controlplane node-role.kubernetes.io/master:NoSchedule-
```

**Output:**
```bash
node/controlplane untainted
```

Adding a `-` at the end of taint removed it.

[Back to TOC](#table-of-contents)

---
### 7. Summary

Taints and tolerations are powerful mechanisms in Kubernetes for controlling pod placement and ensuring that nodes are used efficiently. By applying taints to nodes and corresponding tolerations to pods, you can manage your Kubernetes clusters to meet specific operational requirements. Whether using commands or YAML files, these tools provide flexibility and control over your cluster's behavior.

[Back to TOC](#table-of-contents)
