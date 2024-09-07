### **Tutorial: Consuming Kubernetes Secrets as Environment Variables**

---

### **Table of Contents**

1. [Introduction to Kubernetes Secrets and Environment Variables](#introduction)
2. [Why Use Environment Variables to Consume Secrets?](#why-use-env-variables)
3. [Creating a Secret](#creating-secret)
4. [Consuming Secrets as Environment Variables](#consuming-secret-env)
    - [YAML Example: Injecting a Secret as an Environment Variable](#yaml-example-secret-env)
5. [Verifying Secrets in Pods](#verifying-secret-env)
6. [Security Best Practices for Secrets](#security-best-practices)
7. [Handling Updates to Secrets](#handling-updates)
8. [Summary and Best Practices](#summary)

---

### **1. Introduction to Kubernetes Secrets and Environment Variables** <a name="introduction"></a>

Kubernetes Secrets allow you to store and manage sensitive information, such as passwords, tokens, or SSH keys. Secrets can be consumed by Pods in several ways, one of the most common being via **environment variables**. This tutorial will guide you through consuming Secrets as environment variables in a Kubernetes Pod.

[Back to TOC](#table-of-contents)

---

### **2. Why Use Environment Variables to Consume Secrets?** <a name="why-use-env-variables"></a>

Environment variables provide an easy way to inject sensitive information directly into a container without exposing the data in the application code. This method ensures that:
- Secrets are externalized, making it easier to manage and rotate them.
- Containers can access sensitive information without hardcoding it inside the container image.
- Sensitive information is securely passed to the container at runtime.

Using environment variables is one of the most straightforward ways to access Kubernetes Secrets within an application.

[Back to TOC](#table-of-contents)

---

### **3. Creating a Secret** <a name="creating-secret"></a>

Before consuming a Secret as an environment variable, you need to create a Secret in Kubernetes.

#### **Example: Creating a Secret with Username and Password**

Create a Secret using the `kubectl` command:
```bash
kubectl create secret generic my-db-secret --from-literal=username=admin --from-literal=password=my-password
```

This command creates a Secret named `my-db-secret` with two key-value pairs: `username=admin` and `password=my-password`.

**Verification:**
To verify the Secret was created:
```bash
kubectl get secrets
```

You should see the `my-db-secret` listed.

[Back to TOC](#table-of-contents)

---

### **4. Consuming Secrets as Environment Variables** <a name="consuming-secret-env"></a>

Once the Secret is created, it can be consumed by injecting it into the environment variables of a container running in a Pod.

#### **YAML Example: Injecting a Secret as an Environment Variable** <a name="yaml-example-secret-env"></a>

Here’s how you can inject the Secret we created into a Pod as environment variables:

**Pod Definition:**
```yaml
# pod-with-secret-env.yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
spec:
  containers:
  - name: my-app-container
    image: nginx
    env:
    - name: DB_USERNAME  # Injecting the 'username' key from the Secret
      valueFrom:
        secretKeyRef:
          name: my-db-secret  # The name of the Secret
          key: username  # The key within the Secret
    - name: DB_PASSWORD  # Injecting the 'password' key from the Secret
      valueFrom:
        secretKeyRef:
          name: my-db-secret  # The name of the Secret
          key: password  # The key within the Secret
```

**Explanation:**
- **`env[]`**: Specifies the environment variables for the container.
- **`secretKeyRef`**: References the key-value pairs stored in the Secret `my-db-secret`. The values of `username` and `password` are injected into the environment variables `DB_USERNAME` and `DB_PASSWORD`, respectively.

**Steps to Apply:**
1. Save the above YAML as `pod-with-secret-env.yaml`.
2. Apply the configuration:
   ```bash
   kubectl apply -f pod-with-secret-env.yaml
   ```
   **Expected Output:**
   ```bash
   pod/my-app-pod created
   ```

[Back to TOC](#table-of-contents)

---

### **5. Verifying Secrets in Pods** <a name="verifying-secret-env"></a>

After the Pod has been created, you can verify that the Secrets have been injected into the container's environment variables by running:

**Command:**
```bash
kubectl exec -it my-app-pod -- printenv | grep DB_
```

**Expected Output:**
```
DB_USERNAME=admin
DB_PASSWORD=my-password
```

This confirms that the `DB_USERNAME` and `DB_PASSWORD` environment variables have been set correctly, pulling their values from the `my-db-secret` Secret.

[Back to TOC](#table-of-contents)

---

### **6. Security Best Practices for Secrets** <a name="security-best-practices"></a>

Secrets contain sensitive data, so it's important to follow best practices to ensure they are protected:

#### **Immutability for Secrets**
Secrets should be immutable in production to prevent accidental changes that could lead to service disruption. Once a Secret is marked as immutable, it cannot be modified. If you need to update it, you must create a new Secret.

**Example: Immutable Secret**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-db-secret
  immutable: true  # Mark the Secret as immutable
data:
  username: YWRtaW4=  # Base64-encoded 'admin'
  password: bXktcGFzc3dvcmQ=  # Base64-encoded 'my-password'
```

#### **ReadOnly Volumes**
If you choose to mount Secrets as volumes (covered in another tutorial), always mark them as `readOnly` to prevent accidental modifications to the Secret.

#### **Encryption at Rest**
Although Kubernetes stores Secrets in etcd in Base64-encoded form, it's not true encryption. Enable encryption at rest in etcd to ensure Secrets are secure.

[Back to TOC](#table-of-contents)

---

### **7. Handling Updates to Secrets** <a name="handling-updates"></a>

When Secrets are updated, Pods do not automatically reload the new values. You will need to restart the Pod or rollout the deployment for the updated Secret to take effect in environment variables.

**Restart a Pod to Apply Updated Secrets:**
```bash
kubectl rollout restart deployment <deployment-name>
```

This command forces the Pods to restart and load the updated Secret values as environment variables.

[Back to TOC](#table-of-contents)

---

### **8. Summary and Best Practices** <a name="summary"></a>

In this tutorial, we explored how to consume Kubernetes Secrets as environment variables in a Pod. Using Secrets in this way allows applications to securely access sensitive data without exposing it directly in the codebase.

#### **Key Best Practices:**
- **Immutable Secrets**: Prevent changes to Secrets in production.
- **Pod Restarts**: Remember to restart Pods to apply updated Secret values when using environment variables.
- **Use encryption at rest**: Enable etcd encryption for better protection of Secrets.
  
By following these practices, you can manage sensitive information securely and efficiently in your Kubernetes environment.

[Back to TOC](#table-of-contents)

