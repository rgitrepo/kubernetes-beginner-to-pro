# Kubernetes Metrics Tutorial

## Table of Contents
1. [etcd_server_has_leader](#etcd_server_has_leader)
2. [etcd_server_leader_changes_seen_total](#etcd_server_leader_changes_seen_total)
3. [etcd_network_peer_round_trip_time_seconds_bucket](#etcd_network_peer_round_trip_time_seconds_bucket)
4. [etcd_server_proposals_failed_total](#etcd_server_proposals_failed_total)
5. [workqueue_adds_total](#workqueue_adds_total)
6. [workqueue_depth](#workqueue_depth)
7. [kubelet_running_pods](#kubelet_running_pods)
8. [kubelet_pod_start_duration_seconds_count](#kubelet_pod_start_duration_seconds_count)
9. [kubelet_running_containers](#kubelet_running_containers)
10. [rule_syncs_total](#rule_syncs_total)
11. [apiserver_admission_controller_duration_seconds_bucket](#apiserver_admission_controller_duration_seconds_bucket)
12. [process_cpu_seconds_total](#process_cpu_seconds_total)
13. [apiserver_request_total](#apiserver_request_total)

---

### 1. etcd_server_has_leader
**Explanation**: This metric indicates the current leader of the etcd cluster, the key-value store used by Kubernetes to store cluster data. The leader handles all the writes, and the followers replicate the data.

**Command**:
```shell
curl -s https://<ETCD_SERVER_IP>:2379/metrics | grep etcd_server_has_leader
```

**Sample Output**:
```
etcd_server_has_leader 1
```
**Explanation**: A value of `1` indicates that the etcd server has a leader. A value of `0` indicates no leader, which could cause cluster issues.

[Back to TOC](#table-of-contents)

---

### 2. etcd_server_leader_changes_seen_total
**Explanation**: This metric tracks the number of leader changes seen in the etcd cluster. A high number of changes may indicate instability.

**Command**:
```shell
curl -s https://<ETCD_SERVER_IP>:2379/metrics | grep etcd_server_leader_changes_seen_total
```

**Sample Output**:
```
etcd_server_leader_changes_seen_total 5
```
**Explanation**: This indicates there have been 5 leader changes.

[Back to TOC](#table-of-contents)

---

### 3. etcd_network_peer_round_trip_time_seconds_bucket
**Explanation**: This histogram shows the round-trip time (RTT) between etcd peers, a key measure of network performance.

**Command**:
```shell
curl -s https://<ETCD_SERVER_IP>:2379/metrics | grep etcd_network_peer_round_trip_time_seconds_bucket
```

**Sample Output**:
```
etcd_network_peer_round_trip_time_seconds_bucket{le="0.001"} 5
etcd_network_peer_round_trip_time_seconds_bucket{le="0.002"} 10
etcd_network_peer_round_trip_time_seconds_bucket{le="0.004"} 20
```
**Explanation**: This shows the RTT distribution. For example, there were 5 RTTs ≤ 1ms and 10 RTTs ≤ 2ms.

[Back to TOC](#table-of-contents)

---

### 4. etcd_server_proposals_failed_total
**Explanation**: This metric shows the number of failed proposals in the etcd cluster, indicating potential issues with etcd processing.

**Command**:
```shell
curl -s https://<ETCD_SERVER_IP>:2379/metrics | grep etcd_server_proposals_failed_total
```

**Sample Output**:
```
etcd_server_proposals_failed_total 3
```
**Explanation**: This shows that 3 proposals have failed, which may point to underlying cluster issues that need addressing.

[Back to TOC](#table-of-contents)

---

### 5. workqueue_adds_total
**Explanation**: This metric indicates how many items were added to the workqueue. Kubernetes controllers use workqueues to manage tasks like syncing resources.

**Command**:
```shell
kubectl get --raw /metrics | grep workqueue_adds_total
```

**Sample Output**:
```
workqueue_adds_total{queue="cronjob_controller"} 100
workqueue_adds_total{queue="deployment_controller"} 200
```
**Explanation**: This shows 100 items were added to the `cronjob_controller` queue and 200 to the `deployment_controller` queue.

[Back to TOC](#table-of-contents)

---

### 6. workqueue_depth
**Explanation**: This metric shows the number of items currently waiting to be processed in a workqueue.

**Command**:
```shell
kubectl get --raw /metrics | grep workqueue_depth
```

**Sample Output**:
```
workqueue_depth{queue="cronjob_controller"} 5
workqueue_depth{queue="deployment_controller"} 8
```
**Explanation**: There are 5 items waiting in the `cronjob_controller` queue and 8 in the `deployment_controller` queue.

[Back to TOC](#table-of-contents)

---

### 7. kubelet_running_pods
**Explanation**: This metric shows the number of pods currently running on a kubelet.

**Command**:
```shell
kubectl get --raw /metrics | grep kubelet_running_pods
```

**Sample Output**:
```
kubelet_running_pods 50
```
**Explanation**: This indicates there are 50 running pods on the kubelet.

[Back to TOC](#table-of-contents)

---

### 8. kubelet_pod_start_duration_seconds_count
**Explanation**: This metric counts the total number of times pod start durations have been recorded by the kubelet.

**Command**:
```shell
kubectl get --raw /metrics | grep kubelet_pod_start_duration_seconds_count
```

**Sample Output**:
```
kubelet_pod_start_duration_seconds_count 150
```
**Explanation**: The kubelet has recorded pod start durations 150 times.

[Back to TOC](#table-of-contents)

---

### 9. kubelet_running_containers
**Explanation**: This metric shows how many containers are currently running on the kubelet.

**Command**:
```shell
kubectl get --raw /metrics | grep kubelet_running_containers
```

**Sample Output**:
```
kubelet_running_containers 100
```
**Explanation**: This shows that 100 containers are currently running on the kubelet.

[Back to TOC](#table-of-contents)

---

### 10. rule_syncs_total
**Explanation**: This metric tracks the number of times a rule has been synced, like a Prometheus alert rule.

**Command**:
```shell
kubectl get --raw /metrics | grep rule_syncs_total
```

**Sample Output**:
``>
rule_syncs_total{rule_group="example_rule_group"} 50
```
**Explanation**: The rule group `example_rule_group` has been synced 50 times.

[Back to TOC](#table-of-contents)

---

### 11. apiserver_admission_controller_duration_seconds_bucket
**Explanation**: This histogram tracks the duration for API server admission controllers to process requests.

**Command**:
```shell
kubectl get --raw /metrics | grep apiserver_admission_controller_duration_seconds_bucket
```

**Sample Output**:
```
apiserver_admission_controller_duration_seconds_bucket{le="0.005"} 50
```
**Explanation**: 50 requests were processed in ≤ 5ms.

[Back to TOC](#table-of-contents)

---

### 12. process_cpu_seconds_total
**Explanation**: This metric shows the total CPU time consumed by the Kubernetes process.

**Command**:
```shell
kubectl get --raw /metrics | grep process_cpu_seconds_total
```

**Sample Output**:
```
process_cpu_seconds_total 5000
```
**Explanation**: The Kubernetes process has used 5000 CPU seconds.

[Back to TOC](#table-of-contents)

---

### 13. apiserver_request_total
**Explanation**: This metric tracks the total number of API requests handled by the API server.

**Command**:
```shell
kubectl get --raw /metrics | grep apiserver_request_total
```

**Sample Output**:
```
apiserver_request_total{verb="GET", resource="pods"} 300
apiserver_request_total{verb="POST", resource="deployments"} 150
```
**Explanation**: There have been 300 GET requests for pods and 150 POST requests for deployments.

[Back to TOC](#table-of-contents)

