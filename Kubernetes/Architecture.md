# Kubernetes Architecture

**Kubernetes** is a widely-used open-source platform for managing and deploying containerized applications. Initially developed by Google, it is now maintained by the [Cloud Native Computing Foundation](https://www.cncf.io). Kubernetes automates the deployment, scaling, and management of containerized applications, allowing developers to concentrate on writing code rather than managing infrastructure.

## Key Components

Kubernetes has a modular and scalable architecture, composed of several key components:

- **Kubernetes Master**: Responsible for orchestrating the entire cluster.
- **API Server**: The central management point that facilitates communication between users and the cluster.
- **etcd**: A distributed key-value store that holds the cluster’s configuration data and state.
- **Scheduler**: Allocates resources by assigning workloads to specific nodes.
- **Controller Manager**: Manages the controllers that handle routine tasks and maintain the cluster's desired state.
- **Kubelet**: An agent running on each node, ensuring that the containers are running as expected.
- **Networking Component**: Manages network communication across the cluster.

These components collaborate to provide a powerful platform for managing and deploying containerized applications within a cluster environment.

# Kubernetes Architecture and Components Explained

The **Kubernetes architecture** consists of several components that work together to manage clusters. A **Kubernetes cluster** is a collection of machines, known as **nodes**, which are responsible for running containerized applications. Kubernetes provides a platform that automates the deployment, scaling, and management of these applications.

### Key Concepts

- **Pods**: The smallest and most basic unit of deployment in Kubernetes. A pod is a group of one or more containers that are deployed together on the same node. Pods provide a way to manage containers as a single entity and host the applications running on the cluster.

![Untitled](https://github.com/user-attachments/assets/df1ac1c2-94ba-4759-b244-ac6efb99aa12)


# Control Plane Components

The **Kubernetes control plane** is the central management point for a cluster. It consists of several components that work together to manage the cluster and maintain the desired state of the system.

## Key Components

### kube-apiserver
The **kube-apiserver** is the primary management point for a Kubernetes cluster and one of the core control plane components. It exposes the **Kubernetes API**, which external clients and components use to interact with the cluster.

- Coordinates various cluster components.
- Enforces policies and ensures the desired cluster state is maintained.
- Provides authentication and authorization to ensure only authorized users and applications can access the API.

### etcd
**etcd** is a distributed key-value store used by Kubernetes to store configuration data. It is highly available and consistent, providing a reliable way for different Kubernetes components to store and retrieve cluster state and configuration data.

### kube-scheduler
The **kube-scheduler** is responsible for making scheduling decisions. It assigns workloads to nodes based on several factors, including:

- Resource requirements of the workloads.
- Affinity and anti-affinity rules.
- Data locality and inter-workload interference.
- Hardware, software, or policy constraints, as well as deadlines.

These control plane components work together to ensure the cluster operates efficiently and maintains the desired state as defined by the user.

