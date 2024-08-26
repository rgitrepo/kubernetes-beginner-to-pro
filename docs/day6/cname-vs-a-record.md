# **DNS Record Types: CNAME vs A-Record**

## **Table of Contents**

1. [Introduction](#introduction)
2. [CNAME Record in Kubernetes](#cname-record-in-kubernetes)
   - [CNAME YAML Example for Service](#cname-yaml-example-for-service)
   - [CNAME YAML Example for Ingress](#cname-yaml-example-for-ingress)
3. [A-Record in Kubernetes](#a-record-in-kubernetes)
   - [A-Record YAML Example for Service](#a-record-yaml-example-for-service)
   - [A-Record YAML Example for Ingress](#a-record-yaml-example-for-ingress)
4. [Conclusion](#conclusion)

---

## **Introduction**
In Kubernetes, DNS records play a crucial role in service discovery and external access. Two common types of DNS records used are CNAME (Canonical Name Record) and A-Record (Address Record). 

[Back to TOC](#table-of-contents)

---

## **CNAME Record in Kubernetes**

### **What is a CNAME Record?**
A CNAME record maps one domain name (an alias) to another domain name (the canonical name). This is useful when you want multiple domain names to resolve to the same IP address.

#### **Use Case in Kubernetes:**
CNAME records are typically used when you need to route traffic from a custom domain to a Kubernetes service's external DNS name, such as when using external DNS with cloud services like AWS Route 53 or Google Cloud DNS. The CNAME would point to the Kubernetes service's load balancer DNS name.

### **CNAME YAML Example for Service**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
  namespace: default
  annotations:
    external-dns.alpha.kubernetes.io/hostname: "myapp.example.com"  # The custom domain name
    external-dns.alpha.kubernetes.io/alias: "true"                  # Indicates a CNAME record
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
```

### **CNAME YAML Example for Ingress**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  namespace: default
  annotations:
    external-dns.alpha.kubernetes.io/hostname: "myapp.example.com"  # The custom domain name
    external-dns.alpha.kubernetes.io/alias: "true"                  # CNAME record creation
spec:
  rules:
  - host: "myapp.example.com"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app-service
            port:
              number: 80
```

[Back to TOC](#table-of-contents)

---

## **A-Record in Kubernetes**

### **What is an A-Record?**
An A-Record maps a domain name directly to an IP address (IPv4). It’s a simple DNS record type that provides the actual IP address where the service is hosted.

#### **Use Case in Kubernetes:**
A-Records are often used when you need to directly associate a domain name with the IP address of a Kubernetes service, such as when using static IP addresses for services. This is more common in on-premises environments or when using static IPs with cloud providers.

### **A-Record YAML Example for Service**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
  namespace: default
  annotations:
    external-dns.alpha.kubernetes.io/hostname: "myapp.example.com"  # The custom domain name
spec:
  type: LoadBalancer
  loadBalancerIP: 192.0.2.1                                         # Static IP for the LoadBalancer
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
```

### **A-Record YAML Example for Ingress**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  namespace: default
  annotations:
    external-dns.alpha.kubernetes.io/hostname: "myapp.example.com"  # The custom domain name
spec:
  rules:
  - host: "myapp.example.com"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app-service
            port:
              number: 80
  loadBalancerIP: 192.0.2.1                                         # Static IP for the LoadBalancer
```

[Back to TOC](#table-of-contents)

---

## **Conclusion**

Understanding the differences between CNAME and A-Record is essential when configuring DNS for Kubernetes services. CNAME records provide flexibility by aliasing domain names, while A-Records offer direct mappings to IP addresses. The choice between them depends on your specific use case and infrastructure needs.

[Back to TOC](#table-of-contents)
