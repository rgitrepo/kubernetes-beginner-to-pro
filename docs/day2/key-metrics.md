
# Kubernetes Metrics Tutorial

## Table of Contents
1. [etcd_server_has_leader](#etcd_server_has_leader)
2. [etcd_server_leader_changes_seen_total](#etcd_server_leader_changes_seen_total)
3. [etcd_network_peer_round_trip_time_seconds_bucket](#etcd_network_peer_round_trip_time_seconds_bucket)
4. [workqueue_adds_total](#workqueue_adds_total)
5. [workqueue_depth](#workqueue_depth)
6. [kubelet_running_pods](#kubelet_running_pods)
7. [kubelet_pod_start_duration_seconds_count](#kubelet_pod_start_duration_seconds_count)
8. [rule_syncs_total](#rule_syncs_total)
9. [apiserver_admission_controller_duration_seconds_bucket](#apiserver_admission_controller_duration_seconds_bucket)

---

### 1. etcd_server_has_leader
**Explanation**: This metric indicates the current leader of the etcd cluster, which is a key-value store used by Kubernetes to store all its cluster data. The leader handles all the writes, and the followers replicate the data.

**Example Command**:
```shell
curl -s https://k8s-prod.example.com/metrics | grep etcd_server_has_leader
```

**Sample Output**:
```
etcd_server_has_leader 1
```
**Explanation**: A value of `1` indicates that the etcd server has a leader. A value of `0` would indicate no leader, which would be problematic.

---

### 2. etcd_server_leader_changes_seen_total
**Explanation**: This metric tracks the number of leader changes seen in the etcd cluster. Frequent changes can indicate instability in the cluster.

**Example Command**:
```shell
curl -s https://k8s-prod.example.com/metrics | grep etcd_server_leader_changes_seen_total
```

**Sample Output**:
```
etcd_server_leader_changes_seen_total 5
```
**Explanation**: This indicates there have been 5 leader changes.

---

### 3. etcd_network_peer_round_trip_time_seconds_bucket
**Explanation**: This histogram metric shows the round-trip time (RTT) between etcd peers. RTT is the time it takes for a network request to travel from one peer to another and back.

**Example Command**:
```shell
curl -s https://k8s-prod.example.com/metrics | grep etcd_network_peer_round_trip_time_seconds_bucket
```

**Sample Output**:
```
etcd_network_peer_round_trip_time_seconds_bucket{le="0.001"} 5
etcd_network_peer_round_trip_time_seconds_bucket{le="0.002"} 10
etcd_network_peer_round_trip_time_seconds_bucket{le="0.004"} 20
etcd_network_peer_round_trip_time_seconds_bucket{le="0.008"} 25
etcd_network_peer_round_trip_time_seconds_bucket{le="0.016"} 30
etcd_network_peer_round_trip_time_seconds_bucket{le="0.032"} 35
```
**Explanation**: This output shows the distribution of RTT values. For instance, there were 5 RTTs that were ≤ 1ms, 10 RTTs ≤ 2ms, and so on.

---

### 4. workqueue_adds_total
**Explanation**: This metric indicates the total number of items added to a workqueue. Workqueues are used in Kubernetes to manage tasks such as syncing objects from the API server to controllers.

**Example Command**:
```shell
kubectl get --raw https://k8s-prod.example.com/metrics | grep workqueue_adds_total
```

**Sample Output**:
```
workqueue_adds_total{queue="cronjob_controller"} 100
workqueue_adds_total{queue="deployment_controller"} 200
```
**Explanation**: This shows that 100 items were added to the cronjob_controller queue and 200 to the deployment_controller queue.

---

### 5. workqueue_depth
**Explanation**: This metric indicates the current depth of a workqueue, i.e., how many items are waiting to be processed.

**Example Command**:
```shell
kubectl get --raw https://k8s-prod.example.com/metrics | grep workqueue_depth
```

**Sample Output**:
```
workqueue_depth{queue="cronjob_controller"} 5
workqueue_depth{queue="deployment_controller"} 8
```
**Explanation**: This shows that there are currently 5 items waiting in the cronjob_controller queue and 8 in the deployment_controller queue.

---

### 6. kubelet_running_pods
**Explanation**: This metric shows the number of pods currently running on a kubelet.

**Example Command**:
```shell
kubectl get --raw https://k8s-prod.example.com/metrics | grep kubelet_running_pods
```

**Sample Output**:
```
kubelet_running_pods 50
```
**Explanation**: This indicates there are currently 50 pods running on the kubelet.

---

### 7. kubelet_pod_start_duration_seconds_count
**Explanation**: This metric indicates the count of pod start durations recorded by the kubelet.

**Example Command**:
```shell
kubectl get --raw https://k8s-prod.example.com/metrics | grep kubelet_pod_start_duration_seconds_count
```

**Sample Output**:
```
kubelet_pod_start_duration_seconds_count 150
```
**Explanation**: This indicates that the kubelet has recorded pod start durations 150 times.

---

### 8. rule_syncs_total
**Explanation**: This metric tracks the number of times a rule (e.g., a Prometheus rule) has been synced.

**Example Command**:
```shell
kubectl get --raw https://k8s-prod.example.com/metrics | grep rule_syncs_total
```

**Sample Output**:
```
rule_syncs_total{rule_group="example_rule_group"} 50
```
**Explanation**: This shows that the example_rule_group has been synced 50 times.

---

### 9. apiserver_admission_controller_duration_seconds_bucket
**Explanation**: This histogram metric shows the time taken by the API server’s admission controllers to process requests.

**Example Command**:
```shell
kubectl get --raw https://k8s-prod.example.com/metrics | grep apiserver_admission_controller_duration_seconds_bucket
```

**Sample Output**:
```
apiserver_admission_controller_duration_seconds_bucket{le="0.005"} 50
apiserver_admission_controller_duration_seconds_bucket{le="0.01"} 100
apiserver_admission_controller_duration_seconds_bucket{le="0.025"} 200
apiserver_admission_controller_duration_seconds_bucket{le="0.05"} 300
apiserver_admission_controller_duration_seconds_bucket{le="0.1"} 350
```
**Explanation**: This shows the distribution of admission controller processing times. For example, 50 requests took ≤ 5ms, 100 requests took ≤ 10ms, etc.
