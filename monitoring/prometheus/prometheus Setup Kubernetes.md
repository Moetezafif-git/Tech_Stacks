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

### Prometheus Kubernetes Manifest Files

https://github.com/techiescamp/kubernetes-prometheus
All the configuration files I mentioned in this guide are hosted on Github. You can clone the repo using the following command.
git clone https://github.com/techiescamp/kubernetes-prometheus

### Create a Kubernetes Namespace for Monitoring

To ensure proper organization, we will create a dedicated Kubernetes namespace for all our monitoring components. If you skip this step, all Prometheus deployment objects will be placed in the default namespace.

Run the following command to create a new namespace called `monitoring`:

```code
kubectl create namespace monitoring
````
### Configure RBAC for Prometheus

Prometheus leverages Kubernetes APIs to gather metrics from various resources, including Nodes, Pods, and Deployments. To facilitate this, we need to create a Role-Based Access Control (RBAC) policy that grants read access to the necessary API groups and bind this policy to the `monitoring` namespace.

### Step 1: Create the RBAC Role

Create a file named `clusterRole.yaml` and add the following RBAC role configuration:

**Note:** In the role defined below, we have granted `get`, `list`, and `watch` permissions for `nodes`, `services`, `endpoints`, `pods`, and `ingresses`. The role binding is applied to the `monitoring` namespace. If you need to retrieve metrics from additional resources, you should include those in this cluster role.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus
rules:
- apiGroups: [""]
  resources:
  - nodes
  - nodes/proxy
  - services
  - endpoints
  - pods
  verbs: ["get", "list", "watch"]
- apiGroups:
  - extensions
  resources:
  - ingresses
  verbs: ["get", "list", "watch"]
- nonResourceURLs: ["/metrics"]
  verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: default
  namespace: monitoring
```
### Step 2: Create the RBAC Role

To create the role, execute the following command:

```bash
kubectl create -f clusterRole.yaml

```

### Create a ConfigMap to Externalize Prometheus Configurations

Prometheus configurations are primarily defined in two files:

- **`prometheus.yaml`**: This is the main configuration file for Prometheus, containing all the scrape configurations, service discovery details, storage locations, data retention settings, etc.
- **`prometheus.rules`**: This file contains all the alerting rules for Alertmanager.

By externalizing Prometheus configurations into a Kubernetes ConfigMap, you avoid the need to rebuild the Prometheus image when adding or removing configurations. Instead, simply update the ConfigMap and restart the Prometheus pods to apply the new settings.

The ConfigMap will mount all Prometheus scrape configurations and alerting rules into the Prometheus container at the `/etc/prometheus` location as `prometheus.yaml` and `prometheus.rules`.

### Step 1: Create the ConfigMap YAML File

Create a file named `config-map.yaml` and copy the contents from this link: [Prometheus Config File](#) (insert the actual link here).

### Step 2: Create the ConfigMap in Kubernetes

Execute the following command to create the ConfigMap:

```bash
kubectl create -f config-map.yaml
```
This command creates two files inside the container.

**Note:** In Prometheus terminology, the configuration for collecting metrics from a set of endpoints is referred to as a *job*.

The `prometheus.yaml` file contains configurations to dynamically discover pods and services running in the Kubernetes cluster. The following scrape jobs are defined in our Prometheus scrape configuration:

- **`kubernetes-apiservers`**: Collects metrics from the API servers.
- **`kubernetes-nodes`**: Gathers metrics from all Kubernetes nodes.
- **`kubernetes-pods`**: Discovers metrics from all pods, provided the pod metadata is annotated with `prometheus.io/scrape` and `prometheus.io/port`.
- **`kubernetes-cadvisor`**: Collects all cAdvisor metrics.
- **`kubernetes-service-endpoints`**: Scrapes all service endpoints annotated with `prometheus.io/scrape` and `prometheus.io/port`, which is useful for black-box monitoring.

The `prometheus.rules` file contains all the alert rules for sending alerts to the Alertmanager.

### Create a Prometheus Deployment

#### Step 1: Create the Deployment YAML File

Create a file named `prometheus-deployment.yaml` and copy the following contents into it. In this configuration, we are mounting the Prometheus ConfigMap as a file inside `/etc/prometheus`, as explained in the previous section.

**Note:** This deployment uses the latest official Prometheus image from Docker Hub. Additionally, we are not using any persistent storage volumes for Prometheus storage, as this is a basic setup. When configuring Prometheus for production use cases, ensure you add persistent storage to the deployment.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus-deployment
  namespace: monitoring
  labels:
    app: prometheus-server
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus-server
  template:
    metadata:
      labels:
        app: prometheus-server
    spec:
      containers:
        - name: prometheus
          image: prom/prometheus
          args:
            - "--storage.tsdb.retention.time=12h"
            - "--config.file=/etc/prometheus/prometheus.yml"
            - "--storage.tsdb.path=/prometheus/"
          ports:
            - containerPort: 9090
          resources:
            requests:
              cpu: 500m
              memory: 500Mi
            limits:
              cpu: 1
              memory: 1Gi
          volumeMounts:
            - name: prometheus-config-volume
              mountPath: /etc/prometheus/
            - name: prometheus-storage-volume
              mountPath: /prometheus/
      volumes:
        - name: prometheus-config-volume
          configMap:
            defaultMode: 420
            name: prometheus-server-conf
        - name: prometheus-storage-volume
          emptyDir: {}

```
### Step 2: Create the Deployment in the Monitoring Namespace

Use the following command to create the deployment:

```bash
kubectl create -f prometheus-deployment.yaml
```
### Step 3: Verify the Deployment
You can check the created deployment using the following command:
```bash
kubectl get deployments --namespace=monitoring
```


# Exposing Prometheus Dashboard via Kubernetes Service

To access the Prometheus dashboard via an IP address or DNS name, you need to expose it as a Kubernetes service.

## Step 1: Create a Service YAML File

1. Create a file named `prometheus-service.yaml`.
2. Copy the following contents into the file:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: prometheus-service
  namespace: monitoring
  annotations:
    prometheus.io/scrape: 'true'
    prometheus.io/port: '9090'
spec:
  selector:
    app: prometheus-server
  type: NodePort
  ports:
    - port: 8080
      targetPort: 9090
      nodePort: 30000

```

This configuration will expose Prometheus on all Kubernetes node IPs using port 30000.

## Note
For Cloud Providers (AWS, Azure, Google Cloud): Instead of NodePort, you can set the service type to `LoadBalancer`, which will automatically create a load balancer and point it to the Kubernetes service endpoint.


 ## Step 2: Create the service using the following command.
 
```bash
kubectl create -f prometheus-service.yaml --namespace=monitoring
```
## Step 3: Once created, you can access the Prometheus dashboard using any of the Kubernetes node’s IP on port 30000. If you are on the cloud, make sure you have the right firewall rules to access port 30000 from your workstation.
