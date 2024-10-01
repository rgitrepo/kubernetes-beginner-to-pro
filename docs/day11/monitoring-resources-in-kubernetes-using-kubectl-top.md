### Monitoring Resource Usage in Kubernetes Using `kubectl top`

---

#### Table of Contents
1. [Introduction](#introduction)
2. [Deploying Metrics Server](#deploying-metrics-server)
3. [Identifying the Node Consuming the Most CPU (Cores)](#identifying-the-node-consuming-the-most-cpu-cores)
4. [Identifying the Node Consuming the Most Memory (Bytes)](#identifying-the-node-consuming-the-most-memory-bytes)
5. [Identifying the Pod Consuming the Most Memory (Bytes) in the Default Namespace](#identifying-the-pod-consuming-the-most-memory-bytes-in-the-default-namespace)
6. [Identifying the Pod Consuming the Least CPU (Cores) in the Default Namespace](#identifying-the-pod-consuming-the-least-cpu-cores-in-the-default-namespace)
7. [Outputs Without `--sort-by`](#outputs-without-sort-by)
8. [Conclusion](#conclusion)

---

### Introduction

Monitoring resource usage in Kubernetes clusters is essential to ensure that workloads are efficiently utilizing the available resources. The `kubectl top` command provides insights into CPU and memory consumption for nodes and pods. In this tutorial, we will explore how to deploy the **metrics-server** and then identify the pods and nodes that consume the most or least resources using `kubectl top`.

[Back to TOC](#table-of-contents)

---

### Deploying Metrics Server

Before you can use `kubectl top` to monitor CPU and memory usage, you need to deploy the **metrics-server**, which is responsible for collecting resource metrics from the nodes and pods.

#### Step 1: Clone the Deployment Files

You can pull the deployment files for the metrics-server from a Git repository. Run the following commands:

```bash
git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git
```

#### Step 2: Deploy Metrics Server

Navigate to the cloned repository and apply the configuration files:

```bash
cd kubernetes-metrics-server/
kubectl create -f .
```

**Example Output**:
```plaintext
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
serviceaccount/metrics-server created
deployment.apps/metrics-server created
service/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
```

After successfully deploying the metrics server, you will be able to monitor the resource usage of your nodes and pods.

[Back to TOC](#table-of-contents)

---

### Identifying the Node Consuming the Most CPU (Cores)

To identify which node in your cluster is consuming the most CPU, use the following command:

**Command**:
```bash
kubectl top node --sort-by='cpu' --no-headers | head -1
```

- The `--sort-by='cpu'` flag sorts the nodes in descending order by CPU usage.
- The `--no-headers` flag omits the headers from the output.
- `head -1` prints only the node consuming the most CPU.

**Example Output**:
```plaintext
node01         234m   2%    515Mi   2%
```

In this output:
- `node01` is consuming 234 milli-cores (m) of CPU, which is 2% of its total capacity.
- The memory usage is 515 MiB, which represents 2% of the total memory.

[Back to TOC](#table-of-contents)

---

### Identifying the Node Consuming the Most Memory (Bytes)

To find the node consuming the most memory, you can sort by memory using:

**Command**:
```bash
kubectl top node --sort-by='memory' --no-headers | head -1
```

**Example Output**:
```plaintext
controlplane   183m   2%    662Mi   3%
```

In this example:
- `controlplane` is consuming 662 MiB of memory, which is 3% of its total memory capacity.
- It is also consuming 183 milli-cores of CPU.

[Back to TOC](#table-of-contents)

---

### Identifying the Pod Consuming the Most Memory (Bytes) in the Default Namespace

To determine the pod in the default namespace that is consuming the most memory, run:

**Command**:
```bash
kubectl top pod --namespace=default --sort-by='memory' --no-headers | head -1
```

**Example Output**:
```plaintext
rabbit     134m   201Mi
```

In this output:
- The pod `rabbit` is consuming 201 MiB of memory and 134 milli-cores of CPU.

[Back to TOC](#table-of-contents)

---

### Identifying the Pod Consuming the Least CPU (Cores) in the Default Namespace

To identify the pod consuming the least CPU in the default namespace, use:

**Command**:
```bash
kubectl top pod --namespace=default --sort-by='cpu' --no-headers | tail -1
```

**Example Output**:
```plaintext
lion       2m     5Mi
```

In this output:
- The pod `lion` is consuming 2 milli-cores (m) of CPU, the least amount of CPU in the namespace.
- The memory usage is 5 MiB.

[Back to TOC](#table-of-contents)

---

### Outputs Without `--sort-by`

You can run the `kubectl top` command without the `--sort-by` flag to get the raw output for nodes and pods. This output is not sorted, but it provides a quick glance at the overall resource usage.

#### Example for Nodes:

**Command**:
```bash
kubectl top node
```

**Example Output**:
```plaintext
NAME           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
node01         234m         2%     515Mi           2%
controlplane   183m         2%     662Mi           3%
```

#### Example for Pods in Default Namespace:

**Command**:
```bash
kubectl top pod --namespace=default
```

**Example Output**:
```plaintext
NAME       CPU(cores)   MEMORY(bytes)
rabbit     134m         201Mi
lion       2m           5Mi
```

[Back to TOC](#table-of-contents)

---

### Conclusion

By deploying the **metrics-server** and using the `kubectl top` command, you can monitor the CPU and memory usage of nodes and pods in your Kubernetes cluster. This tutorial demonstrated how to identify the node or pod that consumes the most or least resources, and how to view unsorted outputs for a general overview of resource consumption. Monitoring resource usage helps ensure that your cluster is running efficiently and workloads are distributed optimally.

[Back to TOC](#table-of-contents)
