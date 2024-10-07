![Prometheus-1-800x468](https://github.com/user-attachments/assets/30e40d7f-1141-4db0-ab03-8aa359cbbf3c)

### About Prometheus

Prometheus is a high-scalable open-source monitoring framework. It provides out-of-the-box monitoring capabilities for the Kubernetes container orchestration platform. Also, In the observability space, it is gaining huge popularity as it helps with metrics and alerts.

# Prometheus Overview

| Feature                  | Description                                                                                                                                                   |
|--------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Metric Collection**    | Prometheus employs a **pull model** to collect metrics over HTTP. Metrics can be pushed using **Pushgateway** for scenarios where scraping is not feasible.   |
| **Metric Endpoint**      | Systems must expose metrics through a `/metrics` endpoint for Prometheus to pull them at specified intervals.                                                |
| **PromQL**               | Prometheus features **PromQL**, a flexible query language for querying metrics, used by the Prometheus UI and Grafana for visualization.                      |
| **Prometheus Exporters** | **Exporters** convert metrics from third-party applications into a Prometheus-compatible format. An example is the **Prometheus Node Exporter**, which exposes Linux system metrics. |
| **TSDB (Time-Series Database)** | Prometheus uses a **TSDB** to efficiently store data. Data is stored locally by default, but remote storage options are available to avoid single points of failure. |


### Prometheus Architecture

Here is the high-level architecture of Prometheus. If you want to understand prometheus components in detail, please read the detailed Prometheus Architecture blog.
![prometheus-architecture](https://github.com/user-attachments/assets/bd5f3702-73ea-4e15-b2e1-3c33861ef119)

# The Kubernetes Prometheus monitoring stack has the following components.
1- Prometheus Server
2- Alert Manager
3- Grafana

![kubernetes-1160x564](https://github.com/user-attachments/assets/402d7300-0a15-41c4-84e2-6cb46b28d783)

### Prometheus Monitoring Setup on Kubernetes
  Connect to the Kubernetes Cluster
Connect to your Kubernetes cluster and make sure you have admin privileges to create cluster roles.

Only for GKE: If you are using Google cloud GKE, you need to run the following commands as you need privileges to create cluster roles for this Prometheus setup.
```code
ACCOUNT=$(gcloud info --format='value(config.account)')
kubectl create clusterrolebinding owner-cluster-admin-binding \
    --clusterrole cluster-admin \
    --user $ACCOUNT
````
