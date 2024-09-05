## **ConfigMaps in Kubernetes – Creation and Consumption from Files**

---

### **Table of Contents**
1. [Introduction to ConfigMaps](#introduction-to-configmaps)
2. [Creating a ConfigMap from a File](#creating-a-configmap-from-a-file)
    - [Inspecting the ConfigMap](#inspecting-the-configmap)
3. [Consuming ConfigMap in a Pod](#consuming-configmap-in-a-pod)
    - [Option 1: Using ConfigMap as Environment Variables](#option-1-using-configmap-as-environment-variables)
    - [Option 2: Mounting ConfigMap as Files in a Pod](#option-2-mounting-configmap-as-files-in-a-pod)
4. [Creating ConfigMap from YAML File](#creating-configmap-from-yaml-file)
5. [Visual Output: Screenshots](#visual-output-screenshots)
6. [Summary](#summary)

---

### **1. Introduction to ConfigMaps**

ConfigMaps in Kubernetes allow you to decouple configuration artifacts from image content to keep containerized applications portable. ConfigMaps store key-value pairs which can be consumed by your applications. This tutorial will cover how to create and consume ConfigMaps from files, directories, and directly from YAML files.

[Back to TOC](#table-of-contents)

---

### **2. Creating a ConfigMap from a File**

ConfigMaps can be created from files that contain key-value pairs. Here's an example of a file, `config-file.properties`:

```properties
key1=value1
key2=value2
```

To create a ConfigMap from this file, use the following command:

```bash
kubectl create configmap my-config --from-file=config-file.properties
```

#### **Command Output:**
```bash
configmap/my-config created
```

[Back to TOC](#table-of-contents)

---

### **Inspecting the ConfigMap**

To inspect the ConfigMap and see how it looks in Kubernetes, run:

```bash
kubectl get configmap my-config -o yaml
```

#### **Expected Output:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  config-file.properties: |
    key1=value1
    key2=value2
```

Here, the content of the file is stored as a single entry with key `config-file.properties`.

[Back to TOC](#table-of-contents)

---

### **3. Consuming ConfigMap in a Pod**

There are multiple ways to consume ConfigMaps in Kubernetes Pods. We’ll explore two common methods: using ConfigMaps as environment variables and mounting ConfigMaps as files.

#### **Option 1: Using ConfigMap as Environment Variables**

You can pass key-value pairs from the ConfigMap as environment variables to your container.

##### **Pod YAML File: `pod-env.yaml`**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-env-pod
spec:
  containers:
  - name: my-container
    image: busybox
    command: [ "sh", "-c", "env" ]
    envFrom:
    - configMapRef:
        name: my-config
  restartPolicy: Never
```

To create this pod, run:

```bash
kubectl apply -f pod-env.yaml
```

#### **Verify the ConfigMap Environment Variables:**
Check the logs of the Pod to confirm the environment variables:

```bash
kubectl logs configmap-env-pod
```

#### **Expected Output:**
```
KEY1=value1
KEY2=value2
```

[Back to TOC](#table-of-contents)

---

#### **Option 2: Mounting ConfigMap as Files in a Pod**

Alternatively, you can mount the ConfigMap as a file inside the container.

##### **Pod YAML File: `pod-file.yaml`**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-file-pod
spec:
  containers:
  - name: my-container
    image: busybox
    command: [ "sh", "-c", "cat /etc/config/config-file.properties" ]
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: my-config
  restartPolicy: Never
```

To create this Pod:

```bash
kubectl apply -f pod-file.yaml
```

#### **Verify the ConfigMap Mounted as a File:**
Check the logs of the Pod:

```bash
kubectl logs configmap-file-pod
```

#### **Expected Output:**
```
key1=value1
key2=value2
```

[Back to TOC](#table-of-contents)

---

### **4. Creating ConfigMap from YAML File**

Instead of creating a ConfigMap from the command line, you can define it directly in a YAML file.

##### **ConfigMap YAML File: `my-config.yaml`**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  key1: value1
  key2: value2
```

To apply this ConfigMap:

```bash
kubectl apply -f my-config.yaml
```

#### **Command Output:**
```bash
configmap/my-config created
```

[Back to TOC](#table-of-contents)

---

### **5. Visual Output: Screenshots**

1. **Inspecting the ConfigMap in Kubernetes:**

Here is an example of inspecting the ConfigMap in the terminal:
![ConfigMap inspection output](https://example.com/inspect-configmap-output)

2. **Pod Logs for ConfigMap Mounted as a File:**

Here is an example of checking the logs of a Pod consuming a ConfigMap:
![Pod log output](https://example.com/pod-file-output)

(Note: Replace these URLs with actual screenshots.)

[Back to TOC](#table-of-contents)

---

### **6. Summary**

- **ConfigMaps from files** allow you to load configuration into your Kubernetes resources in a flexible way.
- ConfigMaps can be consumed by Pods either as environment variables or as mounted files.
- YAML definitions give you more control when managing ConfigMaps in your deployment workflows.

This step-by-step tutorial guides you through creating ConfigMaps, inspecting them, and consuming them effectively in Kubernetes Pods.

[Back to TOC](#table-of-contents)

