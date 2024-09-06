### **ConfigMaps - Introduction + Security**

---

### **Table of Contents**

1. [Introduction to ConfigMaps](#introduction)
2. [Application Architectures and ConfigMaps](#application-architectures)
3. [Understanding API Keys and Tokens](#api-keys-tokens)
4. [Security Risks with API Keys](#security-risks)
5. [Why ConfigMaps?](#why-configmaps)
6. [ConfigMaps vs Secrets](#configmaps-vs-secrets)
7. [Creating and Using ConfigMaps](#creating-using-configmaps)
8. [Mutable vs Immutable ConfigMaps](#mutable-immutable)
9. [Real-World Examples](#real-world-example)
10. [Best Practices](#best-practices)
11. [Advanced Configuration: Using ConfigMaps in Containers](#advanced-configuration)
12. [Handling Updates in ConfigMaps](#handling-updates)
13. [Security Considerations](#security-considerations)
14. [Industry Practices for ConfigMaps](#industry-practices)
15. [Consuming ConfigMap Values in a Pod Command](#consuming-configmap-values)

---

### **1. Introduction to ConfigMaps** <a name="introduction"></a>

ConfigMaps in Kubernetes store non-confidential key-value pairs that configure applications, enabling the decoupling of configuration from the application code, making applications more flexible and portable.

[Back to TOC](#table-of-contents)

---

### **2. Application Architectures and ConfigMaps** <a name="application-architectures"></a>

ConfigMaps play a key role in multi-tier applications, managing environment variables and other non-sensitive settings, like frontend-backend configurations, allowing the separation of code from configuration.

**Example:**
A web application backend may use a ConfigMap to store image configurations.

[Back to TOC](#table-of-contents)

---

### **3. Understanding API Keys and Tokens** <a name="api-keys-tokens"></a>

API keys or tokens are used to authenticate requests in applications. These keys should be handled with care due to security risks, which ConfigMaps address by storing non-sensitive API keys.

**Example:**
An API key for a weather service can be stored in a ConfigMap for use by a frontend service.

[Back to TOC](#table-of-contents)

---

### **4. Security Risks with API Keys** <a name="security-risks"></a>

Pushing API keys to version control accidentally is a common risk. These keys should be managed through ConfigMaps or Kubernetes Secrets, rather than hardcoding them into application code.

[Back to TOC](#table-of-contents)

---

### **5. Why ConfigMaps?** <a name="why-configmaps"></a>

ConfigMaps allow you to manage configuration settings separately from code and are particularly useful for non-confidential data like application settings and environment configurations.

**Use Cases:**
- Storing environment variables.
- Configuring image settings in multi-container applications.

[Back to TOC](#table-of-contents)

---

### **6. ConfigMaps vs Secrets** <a name="configmaps-vs-secrets"></a>

While ConfigMaps store non-confidential configuration data, Secrets in Kubernetes handle sensitive information like passwords and tokens, offering encryption and security features.

[Back to TOC](#table-of-contents)

---

### **7. Creating and Using ConfigMaps** <a name="creating-using-configmaps"></a>

#### **Step 1: Creating a ConfigMap**

**From YAML File Example:**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  key1: value1
  key2: value2
```

**Command:**
```bash
kubectl apply -f my-config.yaml
```

#### **Step 2: Configuring ConfigMaps for Use in Pods**

**Using ConfigMap as Environment Variables:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-pod
spec:
  containers:
  - name: env-container
    image: busybox
    command: ["/bin/sh", "-c", "env"]
    env:
    - name: ENV_KEY1
      valueFrom:
        configMapKeyRef:
          name: my-config
          key: key1
    - name: ENV_KEY2
      valueFrom:
        configMapKeyRef:
          name: my-config
          key: key2
```

**Command:**
```bash
kubectl apply -f pod-env.yaml
kubectl logs env-pod
```

**Expected Output:**
```
ENV_KEY1=value1
ENV_KEY2=value2
```

**Using ConfigMap as a Volume (with Read-Only Mount):**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-pod
spec:
  containers:
  - name: volume-container
    image: busybox
    volumeMounts:
    - name: config-volume
      mountPath: "/etc/config"
      readOnly: true  # Read-Only ensures configuration data cannot be modified
    command: ["/bin/sh", "-c", "cat /etc/config/key1 && cat /etc/config/key2"]
  volumes:
  - name: config-volume
    configMap:
      name: my-config
```

**Command:**
```bash
kubectl apply -f pod-volume.yaml
kubectl logs volume-pod
```

**Expected Output:**
```
value1
value2
```

**Explanation:**  
In this example, `readOnly: true` is used to ensure that the mounted ConfigMap is immutable, meaning the container can only read the configuration but cannot modify it.

[Back to TOC](#table-of-contents)

---

### **8. Mutable vs Immutable ConfigMaps** <a name="mutable-immutable"></a>

ConfigMaps can be immutable to ensure they cannot be changed after creation, which is useful for preventing unintended changes in production environments.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
  immutable: true
data:
  key1: value1
  key2: value2
```

[Back to TOC](#table-of-contents)

---

### **9. Real-World Examples** <a name="real-world-example"></a>

#### **Web Application Configuration**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: webapp-config
data:
  APP_ENV: "production"
  API_BASE_URL: "https://api.example.com"
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-pod
spec:
  containers:
  - name: webapp-container
    image: nginx
    env:
    - name: APP_ENV
      valueFrom:
        configMapKeyRef:
          name: webapp-config
          key: APP_ENV
    - name: API_BASE_URL
      valueFrom:
        configMapKeyRef:
          name: webapp-config
          key: API_BASE_URL
```

**Command:**
```bash
kubectl apply -f webapp-pod.yaml
kubectl exec -it webapp-pod -- printenv
```

**Expected Output:**
```
APP_ENV=production
API_BASE_URL=https://api.example.com
```

[Back to TOC](#table-of-contents)

---

### **10. Best Practices** <a name="best-practices"></a>

- Use ConfigMaps to decouple configuration from code.
- For sensitive data, use Secrets instead of ConfigMaps.
- Apply immutability for ConfigMaps in production to prevent accidental changes.
  
[Back to TOC](#table-of-contents)

---

### **11. Advanced Configuration: Using ConfigMaps in Containers** <a name="advanced-configuration"></a>

You can mount ConfigMaps as volumes and read the configuration from files:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app-container
    image: busybox
    volumeMounts:
    - name: config-volume
      mountPath: "/etc/config"
      readOnly: true  # Read-only to prevent modification
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```

**Explanation:**  
The use of `readOnly: true` ensures that the mounted ConfigMap volume cannot be altered, safeguarding configuration data from accidental or unauthorized changes.

[Back to TOC](#table-of-contents)

---

### **12. Handling Updates in ConfigMaps** <a name="handling-updates"></a>

Changes in mounted ConfigMaps automatically propagate without needing to restart Pods. For environment variables, restart Pods to apply updates.

```bash
kubectl rollout restart deployment my-deployment
```

[Back to TOC](#table-of-contents)

---

### **13. Security Considerations** <a name="security-considerations"></a>

Security is crucial when managing ConfigMaps. Here are key security considerations:

#### **1. Mounting ConfigMaps as Read-Only Volumes**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  containers:
  - name: secure-container
    image: nginx
    volumeMounts:
    - name: config-volume
      mountPath: "/etc/config"
      readOnly: true  # Ensures the mounted ConfigMap is read-only
  volumes:
  - name: config-volume
    configMap:
      name: my-config
```

**Explanation:**  
The `readOnly: true` flag ensures that

 the ConfigMap is mounted as a read-only volume, preventing any accidental or malicious changes to the configuration.

---

#### **2. Controlling Access with Kubernetes RBAC**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: configmap-reader
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list"]  # Read-only access to ConfigMaps
```

**Explanation:**  
This RBAC rule restricts access to only reading (`get`, `list`) ConfigMaps, preventing unauthorized modifications.

---

#### **3. Limiting Access to Specific ConfigMaps**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: specific-configmap-reader
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  resourceNames: ["specific-configmap"]  # Access limited to a specific ConfigMap
  verbs: ["get"]
```

**Explanation:**  
Restricting access to a specific ConfigMap ensures that only authorized services can access sensitive configurations.

[Back to TOC](#table-of-contents)

---

### **14. Industry Practices for ConfigMaps** <a name="industry-practices"></a>

- Use clear, purpose-driven names for ConfigMaps.
- Apply immutability for consistency and security.
- Use RBAC to limit access to ConfigMaps.

[Back to TOC](#table-of-contents)

---

### **15. Consuming ConfigMap Values in a Pod Command** <a name="consuming-configmap-values"></a>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: special-pod
spec:
  containers:
  - name: special-container
    image: busybox
    command: ["/bin/echo", "$(SPECIAL_LEVEL_KEY) $(SPECIAL_TYPE_KEY)"]
    env:
    - name: SPECIAL_LEVEL_KEY
      valueFrom:
        configMapKeyRef:
          name: special-config
          key: SPECIAL_LEVEL_KEY
    - name: SPECIAL_TYPE_KEY
      valueFrom:
        configMapKeyRef:
          name: special-config
          key: SPECIAL_TYPE_KEY
```

**Command:**
```bash
kubectl apply -f special-pod.yaml
kubectl logs special-pod
```

**Expected Output:**
```
Hard Boss
```

[Back to TOC](#table-of-contents)

---

This tutorial has been updated to ensure `readOnly` is used where appropriate, especially when mounting ConfigMaps as volumes, to enhance security and prevent modifications. Let me know if you need further examples or explanations!
