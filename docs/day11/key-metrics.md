

# Kubernetes Metrics Tutorial

## Table of Contents
1. [Master Node Metrics](#master-node-metrics)
   1. [etcd_server_has_leader](#1-etcd_server_has_leader)
   2. [etcd_server_leader_changes_seen_total](#2-etcd_server_leader_changes_seen_total)
   3. [etcd_network_peer_round_trip_time_seconds_bucket](#3-etcd_network_peer_round_trip_time_seconds_bucket)
   4. [etcd_server_proposals_failed_total](#4-etcd_server_proposals_failed_total)
   5. [apiserver_request_total](#5-apiserver_request_total)
   6. [apiserver_admission_controller_duration_seconds_bucket](#6-apiserver_admission_controller_duration_seconds_bucket)
   7. [scheduler_e2e_scheduling_duration_seconds_bucket](#7-scheduler_e2e_scheduling_duration_seconds_bucket)
2. [Worker Node Metrics](#worker-node-metrics)
   1. [kubelet_running_pods](#1-kubelet_running_pods)
   2. [kubelet_running_containers](#2-kubelet_running_containers)
   3. [kubelet_pod_start_duration_seconds_count](#3-kubelet_pod_start_duration_seconds_count)
3. [Controller Metrics](#controller-metrics)
   1. [workqueue_adds_total](#1-workqueue_adds_total)
   2. [workqueue_depth](#2-workqueue_depth)
4. [Proxy and Miscellaneous Metrics](#proxy-and-miscellaneous-metrics)
   1. [rule_syncs_total](#1-rule_syncs_total)
   2. [process_cpu_seconds_total](#2-process_cpu_seconds_total)

---

## Master Node Metrics

**Master Node Components**:
- **API Server (port 6443)**: This component is responsible for handling all RESTful calls that update the cluster state.
- **etcd (port 2379)**: A distributed key-value store that holds the cluster data.
- **Controller Manager (port 10257)**: Monitors the state of the cluster and makes sure that the cluster's actual state matches the desired state.
- **Scheduler (optional)**: Decides which node should run a pod.

### Why ports are important
Each Kubernetes component runs on a specific port, which allows them to communicate with each other. For example, API Server listens on port **6443**, etcd on **2379**, and the controller manager on **10257**. Monitoring these components via metrics is essential because they ensure the stability and health of the cluster.

### 1. etcd_server_has_leader
**Explanation**: This metric shows whether the etcd cluster has a leader, which is crucial for maintaining consistency across the cluster.

**Command**:
```shell
curl -s https://<ETCD_SERVER_IP>:2379/metrics | grep etcd_server_has_leader
```

**Sample Output**:
```
etcd_server_has_leader 1
```
**Explanation**: A value of `1` means there is a leader, and `0` means no leader, which can lead to cluster issues.

[Back to TOC](#table-of-contents)

---

### 2. etcd_server_leader_changes_seen_total
**Explanation**: This metric tracks how many times the etcd cluster has seen a change in leadership, which could indicate instability if high.

**Command**:
```shell
curl -s https://<ETCD_SERVER_IP>:2379/metrics | grep etcd_server_leader_changes_seen_total
```

**Sample Output**:
```
etcd_server_leader_changes_seen_total 5
```
**Explanation**: There have been 5 leader changes.

[Back to TOC](#table-of-contents)

---

### 3. etcd_network_peer_round_trip_time_seconds_bucket
**Explanation**: This histogram tracks the round-trip time (RTT) between etcd peers. Longer RTTs indicate network problems.

**Command**:
```shell
curl -s https://<ETCD_SERVER_IP>:2379/metrics | grep etcd_network_peer_round_trip_time_seconds_bucket
```

**Sample Output**:
```
etcd_network_peer_round_trip_time_seconds_bucket{le="0.001"} 5
```
**Explanation**: RTTs of ≤ 1ms occurred 5 times.

[Back to TOC](#table-of-contents)

---

### 4. etcd_server_proposals_failed_total
**Explanation**: Shows the number of failed proposals in etcd, indicating issues with processing.

**Command**:
```shell
curl -s https://<ETCD_SERVER_IP>:2379/metrics | grep etcd_server_proposals_failed_total
```

**Sample Output**:
```
etcd_server_proposals_failed_total 3
```
**Explanation**: 3 proposals have failed.

[Back to TOC](#table-of-contents)

---

### 5. apiserver_request_total
**Explanation**: This tracks the total number of API requests handled by the API server.

**Command**:
```shell
kubectl get --raw /metrics | grep apiserver_request_total
```

**Sample Output**:
```
apiserver_request_total{verb="GET", resource="pods"} 300
```
**Explanation**: There have been 300 GET requests for pods.

[Back to TOC](#table-of-contents)

---

### 6. apiserver_admission_controller_duration_seconds_bucket
**Explanation**: Tracks the time taken by admission controllers to process requests.

**Command**:
```shell
kubectl get --raw /metrics | grep apiserver_admission_controller_duration_seconds_bucket
```

**Sample Output**:
```
apiserver_admission_controller_duration_seconds_bucket{le="0.005"} 50
```
**Explanation**: 50 requests took ≤ 5ms to be processed.

[Back to TOC](#table-of-contents)

---

### 7. scheduler_e2e_scheduling_duration_seconds_bucket
**Explanation**: This metric tracks the scheduling duration of pods from being assigned by the scheduler to the point when they're running.

**Command**:
```shell
kubectl get --raw /metrics | grep scheduler_e2e_scheduling_duration_seconds_bucket
```

**Sample Output**:
```
scheduler_e2e_scheduling_duration_seconds_bucket{le="0.001"} 10
```
**Explanation**: 10 pods were scheduled and running within 1ms.

[Back to TOC](#table-of-contents)

---

## Worker Node Metrics

**Worker Node Components**:
- **Kubelet (port 10250)**: Kubelet is the agent that runs on each worker node. It ensures containers are running in a pod, among other tasks.
- **kube-proxy (optional)**: Manages network connectivity to the services.

### Why ports are important
The kubelet listens on **10250**, a critical port for node operations and metrics collection. Ensuring this port is reachable is key to maintaining proper node health and container orchestration.

### 1. kubelet_running_pods
**Explanation**: Tracks the number of pods running on the kubelet.

**Command**:
```shell
kubectl get --raw /metrics | grep kubelet_running_pods
```

**Sample Output**:
```
kubelet_running_pods 50
```
**Explanation**: 50 pods are currently running.

[Back to TOC](#table-of-contents)

---

### 2. kubelet_running_containers
**Explanation**: Tracks the number of containers running on the kubelet.

**Command**:
```shell
kubectl get --raw /metrics | grep kubelet_running_containers
```

**Sample Output**:
```
kubelet_running_containers 100
```
**Explanation**: There are 100 containers running on this node.

[Back to TOC](#table-of-contents)

---

### 3. kubelet_pod_start_duration_seconds_count
**Explanation**: Counts the number of pod start durations recorded by the kubelet.

**Command**:
```shell
kubectl get --raw /metrics | grep kubelet_pod_start_duration_seconds_count
```

**Sample Output**:
```
kubelet_pod_start_duration_seconds_count 150
```
**Explanation**: Kubelet has recorded pod start durations 150 times.

[Back to TOC](#table-of-contents)

---

## Controller Metrics

**Controller Manager (port 10257)**: This component is responsible for ensuring that the desired state of the system matches the actual state, e.g., restarting a pod if it crashes.

### 1. workqueue_adds_total
**Explanation**: This tracks the number of items added to workqueues.

**Command**:
```shell
kubectl get --raw /metrics | grep workqueue_adds_total
```

**Sample Output**:
```
workqueue_adds_total{queue="cronjob_controller"} 100
```
**Explanation**: 100 items were added to the `cronjob_controller` workqueue.

[Back to TOC](#table-of-contents)

---

### 2. workqueue_depth
**Explanation**: Tracks how many items are waiting to be processed in workqueues.

**Command**:
```shell
kubectl get --raw /metrics | grep workqueue_depth
```

**

Sample Output**:
```
workqueue_depth{queue="cronjob_controller"} 5
```
**Explanation**: 5 items are waiting to be processed in the `cronjob_controller` queue.

[Back to TOC](#table-of-contents)

---

## Proxy and Miscellaneous Metrics

**kube-proxy (optional)**: Handles routing and network traffic for services, but no important metrics are directly tied to kube-proxy by default.

### 1. rule_syncs_total
**Explanation**: This metric tracks how often rules (e.g., Prometheus alert rules) have been synced.

**Command**:
```shell
kubectl get --raw /metrics | grep rule_syncs_total
```

**Sample Output**:
``>
rule_syncs_total{rule_group="example_rule_group"} 50
```
**Explanation**: The rule group `example_rule_group` has synced 50 times.

[Back to TOC](#table-of-contents)

---

### 2. process_cpu_seconds_total
**Explanation**: This metric shows the total CPU time consumed by the Kubernetes process.

**Command**:
```shell
kubectl get --raw /metrics | grep process_cpu_seconds_total
```

**Sample Output**:
```
process_cpu_seconds_total 5000
```
**Explanation**: Kubernetes processes have used 5000 CPU seconds.

[Back to TOC](#table-of-contents)

