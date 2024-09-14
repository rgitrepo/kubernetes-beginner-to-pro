
# Grafana Tutorial

## Table of Contents
1. [Introduction to Grafana](#1-introduction-to-grafana)
2. [Grafana Architecture](#2-grafana-architecture)
3. [Grafana Installation](#3-grafana-installation)
   1. [Installing Grafana Using Helm](#31-installing-grafana-using-helm)
   2. [Manual Grafana Installation](#32-manual-grafana-installation)
4. [Integrating Grafana with Prometheus](#4-integrating-grafana-with-prometheus)
5. [Creating Dashboards in Grafana](#5-creating-dashboards-in-grafana)
   1. [Using Pre-built Dashboards](#51-using-pre-built-dashboards)
   2. [Creating Custom Dashboards](#52-creating-custom-dashboards)
6. [Setting Up Alerts in Grafana](#6-setting-up-alerts-in-grafana)
7. [Advanced Grafana Features](#7-advanced-grafana-features)
8. [Conclusion](#8-conclusion)

---

## 1. Introduction to Grafana
**Grafana** is an open-source platform used for monitoring and observability. It integrates with various data sources like **Prometheus**, **Elasticsearch**, and **InfluxDB** to provide powerful, interactive visualizations.

### Key Features:
- Visualizes time-series data.
- Supports multiple data sources.
- Allows for custom dashboards and alerts.
- Extensible through plugins.

Grafana is widely adopted in **Kubernetes** environments to monitor cluster health, application performance, and custom metrics.

[Back to TOC](#table-of-contents)

---

## 2. Grafana Architecture
Grafana's architecture is built to connect to a variety of **data sources** and create visual dashboards. The key components include:

- **Data Sources**: Where Grafana pulls data from (e.g., Prometheus, InfluxDB).
- **Dashboards**: Visual representations of the metrics.
- **Alerts**: Monitors the data and generates notifications when certain conditions are met.
- **Plugins**: Extend Grafana's functionality by adding custom visualizations or data source integrations.

Grafana uses HTTP requests to fetch data from the backend and generate visualizations for the frontend dashboard.

[Back to TOC](#table-of-contents)

---

## 3. Grafana Installation

### 3.1. Installing Grafana Using Helm
Grafana can be easily deployed in Kubernetes using **Helm**, a package manager for Kubernetes.

#### Steps:
1. **Add the Grafana Helm repository**:
   ```bash
   helm repo add grafana https://grafana.github.io/helm-charts
   ```
2. **Update Helm repositories**:
   ```bash
   helm repo update
   ```
3. **Install Grafana using Helm**:
   ```bash
   helm install grafana grafana/grafana --namespace monitoring
   ```
4. **Expose Grafana**: You can expose Grafana using a LoadBalancer or Ingress.
   ```yaml
   service:
     type: LoadBalancer
   ```

[Back to TOC](#table-of-contents)

---

### 3.2. Manual Grafana Installation
Grafana can also be installed manually on a server or a local machine.

#### Steps:
1. **Download Grafana** from the official website:
   - [https://grafana.com/grafana/download](https://grafana.com/grafana/download)
2. **Install and start Grafana**:
   ```bash
   sudo systemctl start grafana-server
   sudo systemctl enable grafana-server
   ```

Once Grafana is installed, access the dashboard by navigating to `http://<grafana-ip>:3000`. The default username and password are both `admin`.

[Back to TOC](#table-of-contents)

---

## 4. Integrating Grafana with Prometheus
To visualize metrics collected by **Prometheus**, you must first integrate Prometheus as a data source in Grafana.

#### Steps to Add Prometheus as a Data Source:
1. **Login to Grafana**: Navigate to `http://<grafana-ip>:3000` and login.
2. **Go to Configuration** -> **Data Sources**.
3. **Add Data Source** -> Select **Prometheus**.
4. **Configure Prometheus**:
   - Enter the **Prometheus URL** (e.g., `http://<prometheus-ip>:9090`).
5. **Test the connection** to ensure that Grafana can connect to Prometheus.

Grafana will now use Prometheus as a data source to pull metrics for visualizations.

[Back to TOC](#table-of-contents)

---

## 5. Creating Dashboards in Grafana

### 5.1. Using Pre-built Dashboards
Grafana offers **pre-built dashboards** for common use cases, which makes it easier to get started.

#### Steps to Use Pre-built Dashboards:
1. **Go to Dashboard** -> **Manage** -> **Import**.
2. **Enter the Dashboard ID** from Grafana Labs (e.g., for Kubernetes, use ID `6417`).
3. **Load the Dashboard** and select the **Prometheus** data source.
4. The pre-built dashboard will load with default charts for **Kubernetes cluster monitoring**.

[Back to TOC](#table-of-contents)

---

### 5.2. Creating Custom Dashboards
Creating a custom dashboard allows you to define your own metrics and visualizations based on specific needs.

#### Steps:
1. **Create a New Dashboard**:
   - Navigate to **Create** -> **Dashboard** -> **Add New Panel**.
2. **Configure Query**: Use **PromQL** to pull the metrics from Prometheus.
   - Example: `rate(container_cpu_usage_seconds_total[5m])`
3. **Visualize the Data**: Grafana supports different visualizations like **Graphs**, **Bars**, **Pie Charts**, etc.
4. **Save the Dashboard**: Once configured, save your dashboard and make it reusable for future monitoring.

[Back to TOC](#table-of-contents)

---

## 6. Setting Up Alerts in Grafana
Grafana allows setting up **alerts** to notify you when certain conditions are met, such as high CPU usage or low memory.

#### Steps to Set Up Alerts:
1. **Go to the Dashboard** where you want to create an alert.
2. **Click on the Panel** -> **Alert** -> **Create Alert**.
3. **Define the condition** for the alert:
   - Example: Alert if CPU usage exceeds 90%.
4. **Set Notifications**:
   - Grafana supports **Email**, **Slack**, **PagerDuty**, and many other services.
5. **Save the Alert**.

When the alert conditions are met, Grafana will automatically send notifications based on the configured alert rule.

[Back to TOC](#table-of-contents)

---

## 7. Advanced Grafana Features

### 7.1. Annotations
**Annotations** are used to mark important events directly on your Grafana charts. You can use them to highlight when a deployment occurred or an issue was resolved.

### 7.2. Grafana Plugins
Grafana is highly extensible via plugins. You can add new data sources, panels, or even themes via plugins.
- **Example Plugins**: AWS CloudWatch, Zabbix, and MySQL.

### 7.3. Data Source Flexibility
Grafana supports multiple data sources like **Prometheus**, **Elasticsearch**, **AWS CloudWatch**, **Azure Monitor**, and more. This allows for a unified dashboard across various cloud services and on-prem solutions.

[Back to TOC](#table-of-contents)

---

## 8. Conclusion
Grafana is a versatile tool for visualizing metrics in Kubernetes environments. By integrating it with Prometheus, you can create powerful dashboards to monitor the health and performance of your infrastructure. Alerts ensure that you’re notified of critical events, and its wide range of data source support makes Grafana an essential tool in modern observability practices.

[Back to TOC](#table-of-contents)

