## **Creating Kubernetes Secrets

---

### **Table of Contents**

1. [Create Secrets from Literals](#1-create-secrets-from-literals)
2. [Create Secrets from Files](#2-create-secrets-from-files)
3. [Create Secrets from YAML Manifests](#3-create-secrets-from-yaml-manifests)
4. [Create TLS Secrets](#4-create-tls-secrets)
5. [Create Docker Registry Secrets](#5-create-docker-registry-secrets)
6. [Best Practices for Creating Kubernetes Secrets](#6-best-practices)

---

### **1. Create Secrets from Literals** <a name="1-create-secrets-from-literals"></a>

Creating Secrets from literals is one of the quickest ways to store sensitive data in Kubernetes. This method allows you to define key-value pairs directly from the command line.

#### **Command Example**
```bash
kubectl create secret generic my-secret \
  --from-literal=username=admin \
  --from-literal=password=supersecretpassword
```

In this example:
- `my-secret` is the name of the Secret.
- Two key-value pairs (`username=admin` and `password=supersecretpassword`) are created.

#### **Verify the Secret**
After creating the Secret, you can verify its contents with the following command:
```bash
kubectl get secret my-secret -o yaml
```

The output will show the Secret with Base64-encoded values:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: YWRtaW4=  # Base64-encoded 'admin'
  password: c3VwZXJzZWNyZXRwYXNzd29yZA==  # Base64-encoded 'supersecretpassword'
```

This method is useful when you need to quickly create a Secret with simple key-value pairs.

[Back to TOC](#table-of-contents)

---

### **2. Create Secrets from Files** <a name="2-create-secrets-from-files"></a>

When you have sensitive data in a file (e.g., a configuration file, certificate, or key), you can create a Secret directly from that file.

#### **Command Example**
```bash
kubectl create secret generic my-config-secret \
  --from-file=config.json=/path/to/config.json
```

This command:
- Creates a Secret called `my-config-secret` from the `config.json` file.
- The file contents are stored in Base64-encoded format.

#### **Verify the Secret**
You can view the Base64-encoded content of the file:
```bash
kubectl get secret my-config-secret -o yaml
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-config-secret
type: Opaque
data:
  config.json: ZmFrZWNvbmZpZw==  # Base64-encoded content of 'config.json'
```

This method is ideal for storing sensitive configuration files or other data that needs to be securely consumed by your application.

[Back to TOC](#table-of-contents)

---

### **3. Create Secrets from YAML Manifests** <a name="3-create-secrets-from-yaml-manifests"></a>

Creating Secrets declaratively using YAML manifests is a common approach in Infrastructure-as-Code (IaC) environments. This method allows you to define Secrets within a YAML file, which can then be applied using `kubectl`.

#### **YAML Example**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: YWRtaW4=  # Base64-encoded 'admin'
  password: c3VwZXJzZWNyZXRwYXNzd29yZA==  # Base64-encoded 'supersecretpassword'
```

To apply this Secret, use the following command:
```bash
kubectl apply -f my-secret.yaml
```

#### **Why Use YAML Manifests?**
- **Version Control**: YAML files can be tracked in a version control system (e.g., Git) as part of your infrastructure code.
- **Declarative Management**: Manage Secrets in a declarative way, ensuring consistency across environments.

[Back to TOC](#table-of-contents)

---

### **4. Create TLS Secrets** <a name="4-create-tls-secrets"></a>

TLS Secrets are specifically designed to store SSL/TLS certificates and private keys, which are essential for enabling HTTPS in Kubernetes.

#### **Command Example**
```bash
kubectl create secret tls my-tls-secret \
  --cert=/path/to/tls.crt \
  --key=/path/to/tls.key
```

This command:
- Creates a Secret called `my-tls-secret` that stores the TLS certificate (`tls.crt`) and private key (`tls.key`).

#### **YAML Example**
You can also define a TLS Secret using a YAML manifest:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-tls-secret
type: kubernetes.io/tls
data:
  tls.crt: <base64-encoded-cert>
  tls.key: <base64-encoded-key>
```

#### **Use Case**
TLS Secrets are often used in conjunction with Kubernetes Ingress resources to enable HTTPS communication between clients and services.

[Back to TOC](#table-of-contents)

---

### **5. Create Docker Registry Secrets** <a name="5-create-docker-registry-secrets"></a>

Docker Registry Secrets store credentials for pulling container images from private registries. These credentials are stored in a special type of Secret known as `kubernetes.io/dockerconfigjson`.

#### **Command Example**
```bash
kubectl create secret docker-registry my-docker-secret \
  --docker-username=myuser \
  --docker-password=mypassword \
  --docker-email=myemail@example.com \
  --docker-server=myregistry.example.com
```

This command creates a Secret called `my-docker-secret` that stores Docker credentials.

#### **YAML Example**
Alternatively, you can define a Docker registry Secret using a YAML manifest:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-docker-secret
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <base64-encoded-docker-config>
```

#### **Use Case**
Docker registry Secrets are used in Kubernetes Pod specifications to pull images from private registries using the `imagePullSecrets` field.

[Back to TOC](#table-of-contents)

---

### **6. Best Practices for Creating Kubernetes Secrets** <a name="6-best-practices"></a>

When creating Kubernetes Secrets, follow these best practices to ensure security and maintainability:

#### **1. Base64 Encode Sensitive Data**
Always Base64 encode sensitive data before storing it in a Secret manifest. You can do this using the following command:
```bash
echo -n 'my-sensitive-data' | base64
```

#### **2. Avoid Hardcoding Secrets**
Never hardcode sensitive information directly in your application code. Always store it in Secrets and access it securely from your Pods.

#### **3. Use Version Control Cautiously**
If you store Secrets in YAML manifests and commit them to version control, ensure that you are using a private repository and consider encrypting your Secret files.

#### **4. Implement RBAC for Secret Access**
Use Kubernetes Role-Based Access Control (RBAC) to limit which users or services can create, modify, or view Secrets.

#### **5. Use Immutability for Critical Secrets**
For critical Secrets, such as TLS certificates, use the `immutable: true` field to prevent them from being modified after creation.

**Example**:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-immutable-secret
type: Opaque
data:
  key: YWRtaW4=
immutable: true
```

#### **6. Enable Encryption at Rest**
Ensure that your Kubernetes cluster is configured to encrypt Secrets at rest in `etcd` to prevent unauthorized access.

#### **7. Use Tools for Secret Management**
In larger deployments, consider using tools like Sealed Secrets or HashiCorp Vault for better Secret management and security.

[Back to TOC](#table-of-contents)
