
## 4. How to Scrape Metrics

Scraping metrics from a Kubernetes cluster can be done in several ways, depending on what components you want to monitor and how you want to access the data. The three main methods are:

1. **Scraping Directly from Each Component**
2. **Scraping Metrics Directly from the API Server**
3. **Using a Managed Service for Scraping Metrics**

---

### 4.1 Scraping Directly from Each Component

This method involves directly querying the individual components of Kubernetes (e.g., kubelet, etcd, controller manager) for their metrics. This approach provides granular data and flexibility in how you collect metrics.

#### How It Works:
- Each Kubernetes component (such as the API server, etcd, kubelet, and controller manager) exposes its own metrics endpoint.
- Prometheus or another monitoring tool pulls data directly from these components by scraping their HTTP endpoints.

#### Example Components and Endpoints:
- **etcd (port 2379)**: Exposes metrics related to the health and performance of the etcd cluster.
- **API Server (port 6443)**: Provides metrics related to API requests and interactions with the cluster.
- **Kubelet (port 10250)**: Exposes metrics for node-level operations, such as the number of running pods, CPU usage, and memory utilization.

**Why Scrape Directly from Components**:
- **Granularity**: You can target specific components and fine-tune the data you collect.
- **Direct Access**: Avoids potential bottlenecks or single points of failure by bypassing centralized systems.

**Prometheus Scrape Config for Direct Component Scraping**:
```yaml
scrape_configs:
  - job_name: 'kubelet'
    static_configs:
      - targets: ['<KUBELET_IP>:10250']
  - job_name: 'etcd'
    static_configs:
      - targets: ['<ETCD_IP>:2379']
```

**Advantages**:
- Granular control over what metrics to scrape.
- Independent scraping means you can monitor specific components without relying on a single central service.

**Disadvantages**:
- Requires manual configuration of each component.
- Potential for missing metrics if a component is not properly exposed.

[Back to TOC](#table-of-contents)

---

### 4.2 Scraping Metrics Directly from the API Server

Another method is to scrape metrics from the **Kubernetes API Server**. The API server serves as a gateway to access a wide range of metrics related to the cluster’s state and performance. 

#### How It Works:
- The API server exposes a `/metrics` endpoint, which aggregates data from the cluster and its components.
- You can query the API server directly to gather information about resource utilization, API requests, and overall cluster health.

#### Example API Endpoint:
- **API Server Endpoint**: `https://<API_SERVER_IP>:6443/metrics`

#### Why Scrape from the API Server:
- **Unified View**: Scraping the API server gives you a consolidated set of metrics about the state of the entire Kubernetes cluster. This includes both control-plane metrics (such as API request rates) and cluster-level metrics (like the health of nodes and pods).
- **Fewer Endpoints**: Instead of configuring each component individually, you can get an overview by querying the API server.

**Example Prometheus Scrape Config**:
```yaml
scrape_configs:
  - job_name: 'api-server'
    static_configs:
      - targets: ['<API_SERVER_IP>:6443']
```

**Advantages**:
- Centralized source of metrics, reducing the complexity of scraping each component individually.
- Easier to set up compared to scraping individual components.

**Disadvantages**:
- Limited visibility into specific components compared to scraping them directly.
- Reliance on the API server for all metric collection might miss granular details from specific components.

[Back to TOC](#table-of-contents)

---

### 4.3 Using a Managed Service for Scraping Metrics

A **managed service** provides an out-of-the-box solution for monitoring Kubernetes clusters. These services handle the setup, configuration, and management of metrics collection for you, making them an attractive choice for teams that want to focus on application performance rather than managing the monitoring infrastructure.

#### How It Works:
- Managed services like **Datadog**, **New Relic**, and **Splunk** provide integrations for Kubernetes, automatically scraping metrics from various components.
- These services often use agents that run in your cluster to collect data and send it to their centralized monitoring platform.

#### Examples of Managed Services:
- **Datadog**: Collects metrics, logs, and traces from Kubernetes clusters and offers built-in dashboards and alerts.
- **New Relic**: Provides a comprehensive Kubernetes integration for monitoring node health, resource usage, and application performance.
- **Splunk**: Focuses on collecting logs, metrics, and traces from Kubernetes environments and provides advanced search and visualization capabilities.

**Why Use a Managed Service**:
- **Ease of Use**: Managed services handle much of the heavy lifting, from setting up data collection to providing pre-built dashboards and alerts.
- **Scalability**: These services are designed to scale with your infrastructure, handling large amounts of data without requiring additional configuration.

**Advantages**:
- Low operational overhead: Managed services handle scaling, maintenance, and updates.
- Advanced analytics and AI-powered insights often come built-in.

**Disadvantages**:
- Higher costs compared to self-managed solutions like Prometheus.
- Less control over data collection and storage.

**Example Config for Datadog Agent**:
```yaml
apiVersion: v1
kind: DaemonSet
metadata:
  name: datadog-agent
spec:
  selector:
    matchLabels:
      app: datadog-agent
  template:
    metadata:
      labels:
        app: datadog-agent
    spec:
      containers:
      - name: agent
        image: datadog/agent:latest
```

[Back to TOC](#table-of-contents)

---

### Summary of Scraping Methods:

| Method                           | Pros                                       | Cons                                      |
|-----------------------------------|--------------------------------------------|-------------------------------------------|
| **Direct Component Scraping**     | Granular control over individual metrics   | More manual configuration                 |
| **API Server Scraping**           | Simplified setup, central source of truth  | May miss component-specific metrics       |
| **Managed Service**               | Easy setup, scalable, advanced dashboards  | Higher cost, limited control              |

