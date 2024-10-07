Expose an application running in your cluster behind a single outward-facing endpoint, even when the workload is split across multiple backends.

## Kubernetes Services

In Kubernetes, a **Service** serves as a mechanism for exposing a network application that operates through one or more Pods within your cluster.

One of the primary objectives of Services is to eliminate the need for you to alter your existing application to accommodate a new service discovery method. You can run code in Pods, whether it is designed for a cloud-native environment or an older application that you have containerized. A Service facilitates the availability of this set of Pods on the network, allowing clients to interact with them seamlessly.

When you deploy your application using a **Deployment**, it can dynamically create and destroy Pods. This means that at any given moment, the number of active and healthy Pods can vary, and their names may not always be known. Kubernetes manages Pods to match the desired state of your cluster, and because Pods are ephemeral, you should not rely on any individual Pod to be consistent or durable.

Each Pod is assigned a unique IP address, with Kubernetes depending on network plugins to ensure this assignment. As a result, the collection of Pods running a particular Deployment at one moment can differ from the collection of Pods running the same application shortly thereafter.

This variability presents a challenge: if a group of Pods (referred to as "backends") provides services to other Pods (known as "frontends") within the cluster, how do the frontends discover and track the IP addresses they need to connect to in order to interact with the backend services?

This is where **Services** come into play.

## Services in Kubernetes

The **Service API** is an integral part of Kubernetes, providing an abstraction that enables you to expose groups of Pods over a network. Each Service object defines a logical set of endpoints—typically Pods—along with a policy for accessing those Pods.

For instance, consider a stateless image-processing backend running with three replicas. These replicas are interchangeable, meaning that frontend clients do not need to be concerned with which backend they connect to. Even though the actual Pods forming the backend set may change, frontend clients should remain oblivious to these changes and should not have to track the backend set themselves.

The Service abstraction facilitates this decoupling.

The Pods targeted by a Service are usually determined by a selector that you define. To explore other methods for defining Service endpoints, refer to the section on **Services without selectors**.

If your workload uses HTTP, you might opt for an **Ingress** to manage how web traffic is directed to that workload. While Ingress is not a Service type, it functions as the entry point for your cluster. It allows you to consolidate your routing rules into a single resource, enabling you to expose multiple components of your workload—running independently in your cluster—through a single listener.

The **Gateway API** for Kubernetes offers additional capabilities beyond those provided by Ingress and Services. By adding the Gateway to your cluster, which comprises a family of extension APIs implemented using CustomResourceDefinitions, you can configure access to network services running within your cluster.


## Cloud-native Service Discovery

When using Kubernetes APIs for service discovery in your application, you can query the API server for matching **EndpointSlices**. Kubernetes automatically updates the EndpointSlices for a Service whenever the set of Pods associated with that Service changes.

For applications that are not natively designed for Kubernetes, the platform provides options to implement a network port or load balancer between your application and the backend Pods.

Regardless of the approach, your workload can leverage these service discovery mechanisms to locate the target it needs to connect to.
