### Tutorial: Ingress & Ingress Controller

---

### Table of Contents

1. [Introduction to Ingress & Ingress Controller](#introduction-to-ingress-ingress-controller)
2. [Deploying a Load Balancer](#deploying-a-load-balancer)
3. [Static IP vs. Ephemeral IP](#static-ip-vs-ephemeral-ip)
4. [Ingress Overview](#ingress-overview)
5. [Creating and Configuring an Ingress Controller](#creating-and-configuring-an-ingress-controller)
6. [Ingress Architecture](#ingress-architecture)
7. [Path-Based and Domain-Based Routing](#path-based-and-domain-based-routing)
8. [Creating an Ingress Resource](#creating-an-ingress-resource)
9. [Ingress Troubleshooting](#ingress-troubleshooting)
10. [Advanced Topics: Custom Domains and Certificates](#advanced-topics-custom-domains-and-certificates)
11. [Conclusion](#conclusion)

---

### Introduction to Ingress & Ingress Controller
[Back to TOC](#table-of-contents)

Ingress is a key concept in Kubernetes that allows external HTTP and HTTPS traffic to access services within the cluster. An Ingress Controller is responsible for fulfilling the Ingress by routing traffic based on rules defined in the Ingress resource.

---

### Deploying a Load Balancer
[Back to TOC](#table-of-contents)

In Kubernetes, a Load Balancer is typically deployed to manage incoming traffic. However, it's important to note that the IP address of the Load Balancer can change if it is deleted and recreated. This is because Kubernetes assigns an ephemeral IP by default.

**Example:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: MyApp
```

---

### Static IP vs. Ephemeral IP
[Back to TOC](#table-of-contents)

While ephemeral IPs are commonly used, some organizations may opt for a static IP, especially when they require consistent IP addresses for their services. However, using static IPs can be costly and is typically reserved for organizations with specific needs or larger budgets.

---

### Ingress Overview
[Back to TOC](#table-of-contents)

Ingress in Kubernetes acts as a bridge between external requests and the internal services. Unlike a Load Balancer, which might directly expose a service, an Ingress can provide advanced routing and can be used to expose multiple services under a single IP address.

**Key Concepts:**
- **Ingress Resource:** Defines the routing rules.
- **Ingress Controller:** Implements the routing by interpreting the Ingress resource.

---

### Creating and Configuring an Ingress Controller
[Back to TOC](#table-of-contents)

To use Ingress, you must first deploy an Ingress Controller. The Ingress Controller listens to the Ingress resources and fulfills the routing rules.

**Example:**
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

This command installs the NGINX Ingress Controller using a predefined YAML configuration.

---

### Ingress Architecture
[Back to TOC](#table-of-contents)

The Ingress architecture typically involves an end-user, a web application firewall (WAF), a load balancer, and the Ingress Controller. The Ingress Controller handles routing based on the rules specified in the Ingress resource.

**Example Architecture:**
1. **End-User** -> **WAF or Load Balancer** -> **Ingress Controller** -> **Service**

---

### Path-Based and Domain-Based Routing
[Back to TOC](#table-of-contents)

Ingress supports both path-based and domain-based routing, allowing you to direct traffic to different services based on URL paths or hostnames.

**Example:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - host: "example.com"
    http:
      paths:
      - path: /test
        pathType: Prefix
        backend:
          service:
            name: test-service
            port:
              number: 80
```

This Ingress rule routes traffic sent to `example.com/test` to the `test-service` service.

---

### Creating an Ingress Resource
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

### Ingress Troubleshooting
[Back to TOC](#table-of-contents)

Ingress troubleshooting can involve checking logs, ensuring the correct ingress class is defined, and verifying the Ingress resource's configuration.

**Common Issues:**
- **Missing Ingress Class:** Ensure the `ingressClassName` is correctly specified.
- **404 Errors:** This could be due to misconfigured paths or backend services.

**Example:**
```bash
kubectl logs -n ingress-nginx $(kubectl get pods -n ingress-nginx -o name)
```
Check logs for any issues with the Ingress Controller.

---

### Advanced Topics: Custom Domains and Certificates
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

### Conclusion
[Back to TOC](#table-of-contents)

Ingress and Ingress Controllers are powerful tools in Kubernetes that allow for sophisticated traffic management and routing. By understanding how to deploy, configure, and troubleshoot Ingress, you can effectively manage external traffic to your Kubernetes services.

---

This tutorial provides a comprehensive guide to Ingress and Ingress Controllers, covering the essential concepts, configurations, and troubleshooting steps needed to master this critical Kubernetes feature.
