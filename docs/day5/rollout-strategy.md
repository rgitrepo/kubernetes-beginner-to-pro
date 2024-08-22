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
5. [Conclusion](#conclusion)

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

### **Conclusion**
The `RollingUpdate` strategy in Kubernetes allows for seamless application updates with controlled surges and minimal downtime. By understanding and configuring `maxSurge` and `maxUnavailable`, you can fine-tune the deployment process to meet your specific requirements, ensuring a balance between availability and the speed of rollout.

[Back to TOC](#table-of-contents)
