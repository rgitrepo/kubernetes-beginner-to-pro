### **Rollout Strategy with YAML Example**

#### **Table of Contents**
1. [Introduction](#introduction)
2. [YAML File for Rollout Strategy](#yaml-file-for-rollout-strategy)
    - [Metadata Section](#metadata-section)
    - [Labels Section](#labels-section)
    - [Specs Section](#specs-section)
    - [Strategy Section](#strategy-section)
3. [Explanation of YAML Key/Values](#explanation-of-yaml-keyvalues)
4. [YAML Outputs Explained](#yaml-outputs-explained)
5. [Using Rollout Commands](#using-rollout-commands)
    - [Undo](#undo)
    - [Pause and Resume](#pause-and-resume)
    - [--to-revision](#to-revision)
    - [Scale](#scale)
6. [Conclusion](#conclusion)

---

### **Introduction**
In Kubernetes, the deployment strategy defines how updates to applications are rolled out. One of the most commonly used strategies is the rolling update, which allows for updating pods without downtime. This section provides a detailed YAML file example for a rollout strategy, including metadata, labels, specs, and strategy with key values.

[Back to TOC](#table-of-contents)

---

### **YAML File for Rollout Strategy**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rollout-demo
  labels:
    app: nginx
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 50%
      maxUnavailable: 25%
```

[Back to TOC](#table-of-contents)

---

### **Explanation of YAML Key/Values**

#### **Metadata Section**
The `metadata` section contains key information about the deployment:
- `name`: The name of the deployment is `rollout-demo`. This identifier is used throughout the Kubernetes cluster.
- `labels`: A key-value pair (`app: nginx`) that is used to organize and select resources.

#### **Labels Section**
Labels are used to identify and group resources. In this deployment:
- `app: nginx` is applied to the deployment and its pods. These labels are essential for selectors in services, ReplicaSets, and deployments to match the pods they are associated with.

#### **Specs Section**
The `spec` section defines the desired state of the deployment:
- `replicas`: Specifies that 5 pods of `nginx` should be running.
- `selector`: Ensures that the deployment manages pods that have the label `app: nginx`.
- `template`: The pod template contains the configuration for the pod, including the container details:
  - `name`: The container is named `nginx`.
  - `image`: The image used for the container is `nginx:1.14.2`.
  - `ports`: The container exposes port 80.

#### **Strategy Section**
The `strategy` section defines how the deployment should be updated:
- `type`: Specifies `RollingUpdate`, which is the default strategy in Kubernetes. This strategy allows updates to happen incrementally.
- `rollingUpdate`: Defines how the rolling update will proceed:
  - `maxSurge`: 50% - This means that during the update process, up to 50% more pods than the desired number of replicas can be created temporarily. For example, if you have 10 replicas, up to 15 pods may be created during the update.
  - `maxUnavailable`: 25% - This defines that during the update process, a maximum of 25% of the desired replicas can be unavailable at any given time. For 10 replicas, a maximum of 2.5 (rounded to 3) pods can be unavailable during the update.

[Back to TOC](#table-of-contents)

---

### **YAML Outputs Explained**

1. **Deploying the YAML File**
   When this YAML file is applied, Kubernetes will create a deployment named `rollout-demo`. Initially, 5 pods running `nginx:1.14.2` will be created.

   ```bash
   kubectl apply -f rollout-demo.yaml
   ```

   **Output:**

   ```
   deployment.apps/rollout-demo created
   ```

2. **Checking the Deployment Status**
   To check the deployment status and see the rollout in progress, use:

   ```bash
   kubectl rollout status deployment/rollout-demo
   ```

   **Output:**

   ```
   deployment "rollout-demo" successfully rolled out
   ```

3. **Inspecting ReplicaSets**
   Kubernetes will create a new ReplicaSet for the deployment:

   ```bash
   kubectl get rs
   ```

   **Output:**

   ```
   NAME                        DESIRED   CURRENT   READY   AGE
   rollout-demo-7f9d76f8d5     5         5         5       1m
   ```

   This indicates that 5 pods are desired, 5 are currently running, and all 5 are ready.

4. **Scaling the Deployment**
   If you scale the deployment, Kubernetes will handle it according to the `maxSurge` and `maxUnavailable` parameters:

   ```bash
   kubectl scale deployment/rollout-demo --replicas=10
   ```

   **Output:**

   ```
   deployment.apps/rollout-demo scaled
   ```

   The update will proceed with up to 15 pods running (due to `maxSurge: 50%` of 10) and allowing up to 2 or 3 pods to be unavailable at any time.

[Back to TOC](#table-of-contents)

---

### **Using Rollout Commands**

Kubernetes provides several rollout commands to manage deployment updates efficiently. These commands help control the deployment process, handle issues during rollout, and manage the deployment lifecycle.

#### **Undo**

The `kubectl rollout undo` command allows you to revert a deployment to a previous revision. This is useful when a new deployment causes issues, and you need to quickly return to a known stable state.

**Example:**

```bash
kubectl rollout undo deployment/rollout-demo
```

**Output:**

```
deployment.apps/rollout-demo rolled back
```

If you need to roll back to a specific revision, you can use the `--to-revision` flag:

```bash
kubectl rollout undo deployment/rollout-demo --to-revision=2
```

**Reason:** The undo command is essential for disaster recovery. If a deployment introduces bugs or breaks functionality, undoing it to a stable version ensures minimal disruption.

[Back to TOC](#table-of-contents)

#### **Pause and Resume**

The `kubectl rollout pause` command pauses a rollout, stopping the deployment of new pods. This is useful if you want to halt the rollout to investigate an issue or make adjustments.

**Example:**

```bash
kubectl rollout pause deployment/rollout-demo
```

**Output:**

```
deployment.apps/rollout-demo paused
```

To resume the rollout after pausing:

```bash
kubectl rollout resume deployment/rollout-demo
```

**Output:**

```
deployment.apps/rollout-demo resumed
```

**Reason:** Pausing a rollout allows you to temporarily stop the process if issues are detected or if you need to analyze the system before proceeding. It provides control over the deployment process.

[Back to TOC](#table-of-contents)

#### **--to-revision**

The `--to-revision` flag is used with the `kubectl rollout undo` command to revert a deployment to a specific revision.

**Example:**

```bash
kubectl rollout undo deployment/rollout-demo --to-revision=2
```

**Output:**

```
deployment.apps/rollout-demo rolled back to revision 2
```

**Reason:** Sometimes, you need to roll back to a specific previous state rather than the immediate last one. The `--to-revision` flag allows precise control over which deployment revision you revert to, ensuring that you can return to a stable version if necessary.

[Back to TOC](#table-of-contents)

#### **Scale**

The `kubectl scale` command allows you to manually adjust the number of replicas in a deployment. This is often used in response to changing demand or after a rollout.

**Example:**

```bash
kubectl scale deployment/rollout-demo --replicas=3
```

**Output:**

```
deployment.apps/rollout-demo scaled
```

**Reason:** Scaling a deployment is crucial for adjusting the number of running instances to match traffic demand. This command allows you to quickly increase or decrease the number of replicas in response to operational needs.

[Back to TOC](#table-of-contents)

---

### **Conclusion**
Kubernetes rollout commands provide powerful tools for managing and controlling the deployment process. By understanding and using these commands effectively, you can ensure smooth and reliable application updates, recover quickly from issues, and maintain the desired state of your deployments.

[Back to TOC](#table-of-contents)
