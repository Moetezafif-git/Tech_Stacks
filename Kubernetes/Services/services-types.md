## ClusterIP Services

ClusterIP is the default service type in Kubernetes, and it provides internal connectivity between different components of our application. Kubernetes assigns a virtual IP address to a ClusterIP service that can solely be accessed from within the cluster during its creation. This IP address is stable and doesn’t change even if the pods behind the service are rescheduled or replaced.

ClusterIP services are an excellent choice for internal communication between different components of our application that don’t need to be exposed to the outside world. For example, if we have a microservice that processes data and sends it to another microservice for further processing, we can use a ClusterIP service to connect them.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector:
    app: backend
  ports:
  - name: http
    port: 80
    targetPort: 8080
```
## NodePort Services

NodePort services extend the functionality of ClusterIP services by enabling external connectivity to our application. When we create a NodePort service on any node within the cluster that meets the defined criteria, Kubernetes opens up a designated port that forwards traffic to the corresponding ClusterIP service running on the node.

These services are ideal for applications that need to be accessible from outside the cluster, such as web applications or APIs. With NodePort services, we can access our application using the node’s IP address and the port number assigned to the service.
```yaml
apiVersion: v1 
kind: Service 
metadata: 
  name: frontend 
spec: 
  selector: 
    app: frontend 
  type: NodePort 
  ports: 
    - name: http 
      port: 80 
      targetPort: 8080
```
We define a service named frontend that targets pods labeled with app: frontend by setting a selector. The service exposes port 80 and forwards the traffic to the pods’ port 8080. We set the service type to NodePort, and Kubernetes exposes the service on a specific port on a qualifying node within the cluster.

When we create a NodePort service, Kubernetes assigns a port number from a predefined range of 30000-32767. Additionally, we can specify a custom port number by adding the nodePort field to the service definition:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  selector:
    app: frontend
  type: NodePort
  ports:
  - name: http
    port: 80
    targetPort: 8080
    nodePort: 30080
```
The nodePort field is specified as 30080, which tells Kubernetes to expose the service on port 30080 on every node in the cluster.

## LoadBalancer Services
LoadBalancer services connect our applications externally, and production environments use them where high availability and scalability are critical. When we create a LoadBalancer service, Kubernetes provisions a load balancer in our cloud environment and forwards the traffic to the nodes running the service.

LoadBalancer services are ideal for applications that need to handle high traffic volumes, such as web applications or APIs. With LoadBalancer services, we can access our application using a single IP address assigned to the load balancer.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  selector:
    app: web
  type: LoadBalancer
  ports:
    - name: http
      port: 80
      targetPort: 8080
```


| Service Type                          | Use Case                                      | Accessibility                                        | Resource Allocation                       |
|---------------------------------------|-----------------------------------------------|-----------------------------------------------------|------------------------------------------|
| **ClusterIP**                         | Internal communication between application components | Within the cluster only                             | Minimal resources needed                 |
| **NodePort**                          | External accessibility for web applications or APIs | Accessible from outside the cluster via a high-numbered port on the node | Additional resources needed              |
| **LoadBalancer**                      | Production environments with high traffic volumes | Accessible from outside the cluster via a load balancer | Significant resources needed             |
| **Cloud Provider’s Load Balancer**   | Using a cloud provider for Kubernetes         | Accessible from outside the cluster via the cloud provider’s load balancer | May result in cost savings and better performance |
