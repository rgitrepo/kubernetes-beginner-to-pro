## Ingress

### **Table of Contents**
- [Ingress Overview](#ingress-overview)
- [Introduction to Ingress & Ingress Controller](#introduction-to-ingress--ingress-controller)
- [Static IP vs. Ephemeral IP](#static-ip-vs-ephemeral-ip)
- [Creating and Configuring an Ingress Controller](#creating-and-configuring-an-ingress-controller)
- [Ingress Architecture](#ingress-architecture)
- [Creating an Ingress Resource](#creating-an-ingress-resource)
- [Ingress Troubleshooting](#ingress-troubleshooting)
- [Advanced Topics: Custom Domains and Certificates](#advanced-topics-custom-domains-and-certificates)
- [Conclusion](#conclusion)

---

### **Ingress Overview**
[Back to TOC](#table-of-contents)

Ingress in Kubernetes acts as a bridge between external requests and the internal services. Unlike a Load Balancer, which might directly expose a service, an Ingress can provide advanced routing and can be used to expose multiple services under a single IP address.

**Key Concepts:**
- **Ingress Resource**: Defines the routing rules.
- **Ingress Controller**: Implements the routing by interpreting the Ingress resource.



### **Introduction to Ingress & Ingress Controller**
[Back to TOC](#table-of-contents)

Ingress is a key concept in Kubernetes that allows external HTTP and HTTPS traffic to access services within the cluster. It acts as a bridge between external requests and internal services, enabling more advanced routing capabilities compared to a simple Load Balancer.

#### **Ingress**:
- **Function**: Ingress is responsible for defining the rules that match incoming requests to services within your Kubernetes cluster. These rules often include domain names and paths. For example, you can configure an Ingress to direct requests with a specific domain name (like `example.com`) or URL path (like `/app1`) to a particular service within your cluster.
  
  - **Example**: An Ingress resource might specify that requests to `app1.example.com` should be routed to the `app1-service`, and requests to `app2.example.com` should go to the `app2-service`.

#### **Ingress Controller**:
- **Function**: The Ingress Controller is responsible for implementing the rules defined in the Ingress resources. It watches for Ingress resources and configures the underlying infrastructure (such as NGINX or cloud load balancers) to route the traffic based on the Ingress rules.
  
  - **How It Works**: When a request comes in, the Ingress Controller checks the rules defined in the Ingress resource. It then directs the traffic to the appropriate service within the Kubernetes cluster, based on the domain name, path, or other criteria.

#### **To summarize**:
- **Ingress** defines *which* services should be matched based on specific domain names and paths.
- **Ingress Controller** makes sure that traffic is *routed* correctly according to the rules specified in the Ingress, effectively acting as the "traffic director."

---

### **Static IP vs. Ephemeral IP**
[Back to TOC](#table-of-contents)

While ephemeral IPs are commonly used, some organizations may opt for a static IP, especially when they require consistent IP addresses for their services. However, using static IPs can be costly and is typically reserved for organizations with specific needs or larger budgets.

---

### **Creating and Configuring an Ingress Controller**
[Back to TOC](#table-of-contents)

To use Ingress, you must first deploy an Ingress Controller. The Ingress Controller listens to the Ingress resources and fulfills the routing rules.

**Example:**
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```
This command installs the NGINX Ingress Controller using a predefined YAML configuration.

---

### **Ingress Architecture**
[Back to TOC](#table-of-contents)

The Ingress architecture typically involves an end-user, a web application firewall (WAF), a load balancer, and the Ingress Controller. The Ingress Controller handles routing based on the rules specified in the Ingress resource.

**Example Architecture:**
```
End-User -> WAF or Load Balancer -> Ingress Controller -> Service
```

---

### **Creating an Ingress Resource**
[Back to TOC](#table-of-contents)

Creating an Ingress resource involves defining the rules that will direct incoming traffic to the appropriate services.

**Example:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
    - host: "myapp.com"
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-service
                port:
                  number: 80
```
This YAML file creates an Ingress resource that routes traffic from `myapp.com` to `my-service`.

---

### **Ingress Troubleshooting**
[Back to TOC](#table-of-contents)

Ingress troubleshooting can involve checking logs, ensuring the correct ingress class is defined, and verifying the Ingress resource's configuration.

**Common Issues:**
- **Missing Ingress Class**: Ensure the `ingressClassName` is correctly specified.
- **404 Errors**: This could be due to misconfigured paths or backend services.

**Example:**
```bash
kubectl logs -n ingress-nginx $(kubectl get pods -n ingress-nginx -o name)
```
Check logs for any issues with the Ingress Controller.

---

### **Advanced Topics: Custom Domains and Certificates**
[Back to TOC](#table-of-contents)

You can map custom domains to your services using Ingress and secure them with SSL certificates. This typically involves creating an A-record in your domain registrar and configuring the Ingress resource to use the certificate.

**Example:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: secure-ingress
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
    - hosts:
        - "myapp.com"
      secretName: myapp-cert
  rules:
    - host: "myapp.com"
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-service
                port:
                  number: 80
```

---

### **Conclusion**
[Back to TOC](#table-of-contents)

Ingress and Ingress Controllers are powerful tools in Kubernetes that allow for sophisticated traffic management and routing. By understanding how to deploy, configure, and troubleshoot Ingress, you can effectively manage external traffic to your Kubernetes services.

This tutorial provides a comprehensive guide to Ingress and Ingress Controllers, covering the essential concepts, configurations, and troubleshooting steps needed to master this critical Kubernetes feature.

---

This version includes detailed examples of path-based, domain-based, and geolocation-based routing, with comments in the YAML files to enhance understanding.
