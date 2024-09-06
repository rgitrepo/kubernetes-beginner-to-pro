### **Tutorial: Consuming ConfigMaps Using Environment Variables in Kubernetes**

---

### **Table of Contents**

1. [Introduction to ConfigMaps and Environment Variables](#introduction)
2. [Why Use Environment Variables for ConfigMap Consumption?](#why-use-environment-variables)
3. [Creating a ConfigMap](#creating-configmap)
4. [Consuming ConfigMap Values as Environment Variables](#consuming-configmap)
    - [YAML Example: Injecting Environment Variables from a ConfigMap](#yaml-example-injecting-env)
5. [Verifying Environment Variables in Pods](#verifying-environment-variables)
6. [Handling Updates in ConfigMaps](#handling-updates)
7. [Summary and Best Practices](#summary)

---

### **1. Introduction to ConfigMaps and Environment Variables** <a name="introduction"></a>

Kubernetes ConfigMaps store key-value pairs for configuration purposes, allowing applications to externalize configuration data instead of hardcoding it within the application itself. One of the primary ways to consume ConfigMap data is by injecting it as **environment variables** into the Pods where the application is running.

Using environment variables provides flexibility, making it easier to change application behavior across environments (e.g., development, staging, production) without altering the code or redeploying images.

[Back to TOC](#table-of-contents)

---

### **2. Why Use Environment Variables for ConfigMap Consumption?** <a name="why-use-environment-variables"></a>

Environment variables are a lightweight and convenient way to pass configuration data to containers. Some common reasons to use environment variables for ConfigMap consumption include:
- **Ease of access**: Applications can access environment variables without needing to manage files or directories.
- **Dynamic configuration**: You can easily swap configurations by modifying the ConfigMap without redeploying the container.
- **Portability**: Environment variables help decouple configuration from the application, making it portable across multiple environments (e.g., development, QA, production).

[Back to TOC](#table-of-contents)

---

### **3. Creating a ConfigMap** <a name="creating-configmap"></a>

To begin, create a ConfigMap that stores key-value pairs. This ConfigMap will later be consumed as environment variables in your Pods.

**Example ConfigMap:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: default
data:
  APP_ENV: "production"
  API_URL: "https://api.example.com"
  MAX_RETRIES: "5"
```

**Explanation:**
- **`APP_ENV`**: Sets the environment to "production."
- **`API_URL`**: Specifies the base URL for the application API.
- **`MAX_RETRIES`**: Defines how many times the application should retry failed requests.

**Create the ConfigMap:**
```bash
kubectl apply -f app-config.yaml
```

This will create a ConfigMap named `app-config` in the default namespace.

[Back to TOC](#table-of-contents)

---

### **4. Consuming ConfigMap Values as Environment Variables** <a name="consuming-configmap"></a>

Once the ConfigMap is created, you can inject its values as environment variables into a Pod. This allows the application running in the Pod to access these values as standard environment variables.

#### **YAML Example: Injecting Environment Variables from a ConfigMap** <a name="yaml-example-injecting-env"></a>

Here’s how you can configure a Pod to use environment variables from the ConfigMap we created earlier.

**Pod Definition:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
  namespace: default
spec:
  containers:
  - name: app-container
    image: nginx
    env:
    - name: APP_ENV  # Injects the APP_ENV key from the ConfigMap
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_ENV
    - name: API_URL  # Injects the API_URL key from the ConfigMap
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: API_URL
    - name: MAX_RETRIES  # Injects the MAX_RETRIES key from the ConfigMap
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: MAX_RETRIES
```

**Explanation:**
- **`env[]`**: The environment variable section in the Pod spec.
- **`configMapKeyRef`**: Refers to a specific key in the `app-config` ConfigMap. Each key-value pair in the ConfigMap is injected as an environment variable in the container.
- **`name`**: Defines the environment variable name that will be available to the application.

**Deploy the Pod:**
```bash
kubectl apply -f app-pod.yaml
```

This will create a Pod named `app-pod` with the environment variables injected from the ConfigMap.

[Back to TOC](#table-of-contents)

---

### **5. Verifying Environment Variables in Pods** <a name="verifying-environment-variables"></a>

Once the Pod is running, you can verify that the environment variables have been injected correctly by checking the environment variables inside the container.

**Command:**
```bash
kubectl exec -it app-pod -- printenv
```

**Expected Output:**
```
APP_ENV=production
API_URL=https://api.example.com
MAX_RETRIES=5
```

The output shows that the environment variables `APP_ENV`, `API_URL`, and `MAX_RETRIES` are set with the values from the ConfigMap.

[Back to TOC](#table-of-contents)


---

### **6. Handling Updates in ConfigMaps** <a name="handling-updates"></a>

When you update a ConfigMap, the changes do not automatically propagate to Pods using the ConfigMap as environment variables. 

You need to **restart** the Pod to apply the updated environment variables. Update is automatic in case of Volumes.

Almost always try to use volume mounts and avoid environment variables.

**Command to Restart Pods:**
```bash
kubectl rollout restart deployment <deployment-name>
```

This will recreate the Pods, injecting the updated ConfigMap values as environment variables.

[Back to TOC](#table-of-contents)

---

### **7. Summary and Best Practices** <a name="summary"></a>

In this tutorial, we covered how to consume ConfigMap values as environment variables in Kubernetes. Using environment variables provides a convenient way to externalize configuration while keeping application code portable and environment-independent.

#### **Key Best Practices:**
- **Use Immutability**: Ensure that production ConfigMaps are immutable to prevent accidental changes.
- **Security**: Use read-only mounts when consuming ConfigMaps as volumes.
- **Restart Pods**: Be aware that changes in ConfigMaps require a Pod restart to take effect when consumed as environment variables.

This method is ideal for small-to-medium applications that need dynamic configuration management without complex file handling.

[Back to TOC](#table-of-contents)

---

Would you like additional examples or details on using environment variables with ConfigMaps in more complex scenarios?
