
# Prometheus Tutorial for Kubernetes

## Table of Contents
1. [Introduction to Prometheus](#introduction-to-prometheus)
2. [How Prometheus Works](#how-prometheus-works)
3. [Prometheus Architecture in Kubernetes](#prometheus-architecture-in-kubernetes)
4. [Scraping Metrics in Prometheus](#scraping-metrics-in-prometheus)
   1. [Pull-Based Model](#pull-based-model)
   2. [Push Gateway](#push-gateway)
   3. [Three Ways to Scrape Metrics](#three-ways-to-scrape-metrics)
5. [Prometheus Query Language (PromQL)](#prometheus-query-language-promql)
6. [Setting Up Alerts in Prometheus](#setting-up-alerts-in-prometheus)
   1. [Prometheus Alertmanager](#prometheus-alertmanager)
7. [Prometheus Integration with Grafana](#prometheus-integration-with-grafana)
8. [Use Cases of Prometheus in Kubernetes](#use-cases-of-prometheus-in-kubernetes)
9. [Best Practices for Using Prometheus](#best-practices-for-using-prometheus)
10. [Conclusion](#conclusion)

---

## 1. Introduction to Prometheus
**Prometheus** is an open-source monitoring system widely used in Kubernetes to collect and store metrics as time-series data. It is designed to monitor dynamic environments, making it ideal for container orchestration platforms like Kubernetes.

Prometheus operates by pulling metrics from various sources, storing them in a time-series database (TSDB), and supporting flexible querying with PromQL.

### Key Features:
- Pull-based metric collection.
- Time-series storage.
- Alerting capabilities.
- Integration with Grafana for visualization.

[Back to TOC](#table-of-contents)

---

## 2. How Prometheus Works
Prometheus follows a **pull-based** model where it regularly scrapes predefined endpoints to gather metrics. It scrapes at specified intervals and stores the data as time-series metrics. This data can then be queried using **PromQL**, the native query language of Prometheus.

Prometheus works well with Kubernetes due to its ability to monitor:
- **Infrastructure metrics** (nodes, CPU, memory, disk usage).
- **Application metrics** (availability, response times).
- **Custom metrics** (defined by the developer).

[Back to TOC](#table-of-contents)

---

## 3. Prometheus Architecture in Kubernetes
Prometheus integrates into the Kubernetes ecosystem using the following components:

1. **Prometheus Server**: Responsible for scraping metrics from targets, storing data, and providing an API for querying.
2. **Alertmanager**: Handles alerts sent by Prometheus and can forward them to email, Slack, or other services.
3. **Push Gateway**: Used to collect metrics from short-lived jobs (e.g., batch jobs) that do not expose metrics endpoints.
4. **Grafana**: A dashboarding tool for visualizing Prometheus metrics.

Each component can be deployed as a **Kubernetes pod** using Helm charts or custom configurations.

### Prometheus Ports in Kubernetes:
- **API server (6443)**: Prometheus scrapes metrics from the Kubernetes API server.
- **etcd (2379)**: Prometheus scrapes metrics from etcd.
- **Controller Manager (10257)**: Prometheus scrapes metrics from the controller manager.
- **Kubelet (10250)**: Prometheus scrapes metrics from worker nodes using the kubelet.

[Back to TOC](#table-of-contents)

---

## 4. Scraping Metrics in Prometheus

### 4.1. Pull-Based Model
Prometheus operates on a **pull-based model**, where it retrieves metrics from applications by periodically scraping specified endpoints. This is preferred over the push model as it provides better control over which metrics are collected and when.

### 4.2. Push Gateway
For **short-lived jobs** or batch processes, Prometheus can use the **Push Gateway**. This allows these ephemeral processes to push their metrics before they are terminated.

Use cases:
- **Batch Jobs**: Push metrics before the job terminates.
- **Ephemeral Pods**: When pods complete and are removed from the cluster, their metrics are retained.

### 4.3. Three Ways to Scrape Metrics
1. **Directly from each component**: Prometheus scrapes metrics from each Kubernetes component like kubelet, API server, etcd.
   - Example: Scraping kubelet metrics directly via the `10250` port.
2. **Scrape via API server**: Prometheus scrapes metrics from the API server, which aggregates metrics from other components.
   - Example: API server port `6443` provides a central point for metric collection.
3. **Using a managed service**: Prometheus can be configured to work with managed Kubernetes services like AWS EKS, GCP GKE, or Azure AKS. In these cases, Prometheus scrapes data via the service provider's endpoints.

[Back to TOC](#table-of-contents)

---

## 5. Prometheus Query Language (PromQL)
**PromQL** is the query language used by Prometheus to retrieve metrics. It allows users to filter, aggregate, and analyze time-series data stored in Prometheus.

### Example Queries:
- **CPU usage over time**:
  ```promql
  rate(node_cpu_seconds_total[5m])
  ```
- **Memory usage**:
  ```promql
  node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes * 100
  ```

PromQL provides functions to perform calculations, aggregations, and create visualizations for better insights into the data.

[Back to TOC](#table-of-contents)

---

## 6. Setting Up Alerts in Prometheus

### 6.1. Prometheus Alertmanager
**Prometheus Alertmanager** handles alert notifications sent from the Prometheus server. It can group, route, and deliver alerts to multiple destinations such as:
- Email
- Slack
- PagerDuty
- Webhooks

### 6.2. Setting Up Alerts
1. Define **alerting rules** in Prometheus.
   Example:
   ```yaml
   groups:
     - name: pod-memory
       rules:
         - alert: HighMemoryUsage
           expr: sum(container_memory_usage_bytes{container_name!=""}) > 500Mi
           for: 5m
           labels:
             severity: critical
           annotations:
             summary: "High memory usage detected in pod"
   ```
2. Configure **Alertmanager** to receive and forward alerts:
   ```yaml
   global:
     resolve_timeout: 5m
   receivers:
     - name: 'team-email'
       email_configs:
         - to: 'team@example.com'
   ```

3. Use **Grafana** to visualize alerts.

[Back to TOC](#table-of-contents)

---

## 7. Prometheus Integration with Grafana
**Grafana** is a powerful visualization tool that integrates seamlessly with Prometheus to create interactive dashboards.

### Steps to Integrate:
1. **Install Grafana**: Deploy it using Helm or manually via YAML.
2. **Add Prometheus as a data source**:
   - In Grafana, go to **Configuration** -> **Data Sources** -> **Add Prometheus**.
   - Provide the URL of the Prometheus server (e.g., `http://<prometheus-server>:9090`).
3. **Create Dashboards**: Use pre-built templates or create custom dashboards to visualize metrics like CPU, memory, or custom application data.

[Back to TOC](#table-of-contents)

---

## 8. Use Cases of Prometheus in Kubernetes

1. **Cluster Health Monitoring**: Track node health, pod availability, and system resources like CPU and memory.
2. **Application Performance Monitoring**: Monitor application-level metrics (e.g., request rates, error rates).
3. **Custom Business Metrics**: Track custom metrics, such as the number of processed transactions or API response times.

Prometheus is widely used in production environments to monitor both infrastructure and application metrics, ensuring visibility into the entire system.

[Back to TOC](#table-of-contents)

---

## 9. Best Practices for Using Prometheus
- **Configure proper scraping intervals**: Choose appropriate scraping intervals based on the metric’s importance. Critical metrics may require shorter intervals. Normally intervals are between 30-60s. For financial applications they are around 10-15s.
- **Use labels effectively**: Leverage labels to categorize and aggregate metrics in Prometheus queries.
- **Alert on actionable metrics**: Avoid alert fatigue by setting up alerts only on critical issues that require human intervention.
- **Leverage Grafana for visualization**: Combine Prometheus with Grafana for effective visualizations.
- **Consider scaling Prometheus**: Use **Thanos** or **Cortex** to scale Prometheus in larger environments where multiple clusters are monitored.

[Back to TOC](#table-of-contents)

---

## 10. Conclusion
Prometheus is an essential tool for monitoring and alerting in Kubernetes environments. By integrating it with Grafana and using best practices, you can gain deep insights into your infrastructure and applications, ensuring they run smoothly and efficiently.

This tutorial covered the basics of Prometheus architecture, how to scrape metrics, set up alerts, and integrate with Grafana for visualizations.

[Back to TOC](#table-of-contents)
