
## Table of Contents

1. [Kubernetes Controller Manager Overview](#kubernetes-controller-manager-overview)
2. [General Overview](#general-overview)
3. [Controllers](#controllers)
   - [Node Controller](#1-node-controller)
   - [Service Controller](#2-service-controller)
   - [Namespace Controller](#3-namespace-controller)
   - [DaemonSet Controller](#4-daemonset-controller)
   - [CronJob Controller](#5-cronjob-controller)
   - [Custom Controller](#6-custom-controller)
4. [Conclusion](#conclusion)

## Kubernetes Controller Manager Overview

The Kubernetes Controller Manager is a key component of the Kubernetes control plane. It is responsible for running controllers that regulate the state of the Kubernetes cluster. Each controller watches the state of the cluster through the API server and makes necessary changes to move the current state towards the desired state.

<div style="text-align: center;">
  <img src="../../pics/kube-controller-manager.gif" alt="Kube-Scheduler style="width: 600px; height: 450px;">
</div>

## General Overview

1. **Purpose**: The Controller Manager ensures that the cluster is always in the desired state. It does this by running a series of controllers that perform routine tasks, such as creating pods, maintaining node status, and managing endpoints.
2. **Functionality**: Each controller manages a specific aspect of the cluster. They watch the state of objects through the API server and take action to achieve the desired state.
3. **Execution**: Controllers can be run as individual processes, but to simplify deployment and reduce overhead, they are typically compiled into a single binary and run in a single process.

## Controllers

### 1. Node Controller

**Purpose**: Manages the state of nodes in the cluster.

- **Responsibilities**:
  - Monitors the health of nodes.
  - Detects node failures and updates the node status.
  - Initiates the eviction of pods from a node if it becomes unreachable.

- **Details**:
  - **Node Monitoring**: Periodically checks the status of each node.
  - **Node Eviction**: If a node does not respond within a specified time, it marks the node as "NotReady" and starts pod eviction.

### 2. Service Controller

**Purpose**: Manages the state of services in the cluster.

- **Responsibilities**:
  - Ensures that the service endpoints are correctly configured.
  - Manages cloud provider-specific load balancers.

- **Details**:
  - **Endpoint Management**: Watches services and endpoints objects, ensuring that the service’s endpoints match the expected state.
  - **Load Balancers**: Interacts with the cloud provider API to create, update, or delete load balancers.

### 3. Namespace Controller

**Purpose**: Manages namespaces in the cluster.

- **Responsibilities**:
  - Ensures that the lifecycle of all objects within a namespace is managed correctly.
  - Handles the cleanup of resources when a namespace is deleted.

- **Details**:
  - **Namespace Deletion**: When a namespace is deleted, it ensures that all related resources (pods, services, etc.) are also deleted.
  - **Resource Cleanup**: Cleans up leftover resources that might have been stuck due to failures or partial deletions.

### 4. DaemonSet Controller

**Purpose**: Manages DaemonSets, ensuring that a daemon pod runs on every node (or a subset of nodes).

- **Responsibilities**:
  - Schedules daemon pods on nodes.
  - Ensures that pods are created and maintained as specified.

- **Details**:
  - **Pod Scheduling**: Ensures that a daemon pod is running on all eligible nodes.
  - **Pod Management**: Monitors the health of daemon pods and recreates them if they fail.

### 5. CronJob Controller

**Purpose**: Manages CronJobs, which run jobs on a scheduled basis.

- **Responsibilities**:
  - Ensures that jobs are created and run according to the specified schedule.
  - Handles job failures and retries.

- **Details**:
  - **Scheduling**: Uses Cron syntax to schedule jobs.
  - **Job Execution**: Creates Job objects based on the schedule and manages their lifecycle.

### 6. Custom Controller

**Purpose**: Manages custom resources and their lifecycle.

- **Responsibilities**:
  - Allows users to define custom resources and controllers to manage them.
  - Integrates with the Kubernetes API to provide custom functionality.

- **Details**:
  - **Custom Resources**: Allows for the extension of Kubernetes API with new resource types.
  - **Custom Logic**: Users can write their own controllers to manage the lifecycle and state of these custom resources.

## Conclusion

The Kubernetes Controller Manager is a vital component of the Kubernetes control plane, responsible for maintaining the desired state of the cluster by running a set of controllers. Each controller focuses on managing specific resources and ensuring that their state matches the declared specifications. By handling tasks like node health monitoring, service endpoint management, and job scheduling, the Controller Manager helps maintain the robustness and reliability of the Kubernetes cluster.


