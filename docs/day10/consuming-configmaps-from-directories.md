### **Tutorial: Consuming ConfigMaps from Directories in Kubernetes**

---

### **Table of Contents**
1. [Introduction to ConfigMaps from Directories](#introduction-to-configmaps-from-directories)
2. [Creating a ConfigMap from a Directory](#creating-a-configmap-from-a-directory)
    - [Inspecting the ConfigMap](#inspecting-the-configmap)
3. [Consuming ConfigMap in a Pod](#consuming-configmap-in-a-pod)
    - [Option 1: Mounting ConfigMap as Files in a Pod](#option-1-mounting-configmap-as-files-in-a-pod)
    - [Option 2: Using Specific Keys as Environment Variables](#option-2-using-specific-keys-as-environment-variables)
4. [Summary](#summary)

---

### **1. Introduction to ConfigMaps from Directories**

Kubernetes allows you to create ConfigMaps from directories where each file in the directory represents a key, and the content of the file becomes the value. This approach is useful when you have multiple configuration files you want to manage together in a single ConfigMap. In this tutorial, we will walk through the process of creating a ConfigMap from a directory and consuming it in a Pod.

[Back to TOC](#table-of-contents)

---

### **2. Creating a ConfigMap from a Directory**

To create a ConfigMap from a directory, each file in the directory will act as a key, and its contents will be used as the value.

#### **Directory Structure Example:**
```
config-dir/
├── key1.txt
└── key2.txt
```

##### **Content of `key1.txt`:**
```text
value1
```

##### **Content of `key2.txt`:**
```text
value2
```

You can create the ConfigMap from this directory using the following command:

```bash
kubectl create configmap my-config --from-file=config-dir/
```

#### **Command Output:**
```bash
configmap/my-config created
```

[Back to TOC](#table-of-contents)

---

### **Inspecting the ConfigMap**

To see how the created ConfigMap looks in Kubernetes, run:

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
  key1.txt: value1
  key2.txt: value2
```

Here, the keys are the filenames (`key1.txt` and `key2.txt`), and their contents (`value1` and `value2`) are the respective values.

[Back to TOC](#table-of-contents)

---

### **3. Consuming ConfigMap in a Pod**

Once the ConfigMap is created, it can be consumed by a Pod in two main ways:

#### **Option 1: Mounting ConfigMap as Files in a Pod**

In this option, each file in the ConfigMap is mounted as a file inside the container.

##### **Pod YAML File: `pod-mount-dir.yaml`**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-mount-dir-pod
spec:
  containers:
  - name: my-container
    image: busybox
    command: [ "sh", "-c", "cat /etc/config/key1.txt /etc/config/key2.txt" ]
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: my-config
  restartPolicy: Never
```

This Pod mounts the ConfigMap `my-config` into the `/etc/config` directory inside the container. The keys `key1.txt` and `key2.txt` become files inside this directory.

##### **Command to Apply Pod YAML:**
```bash
kubectl apply -f pod-mount-dir.yaml
```

#### **Check the Logs to Verify:**
You can check the logs of the Pod to confirm that the files have been mounted correctly:

```bash
kubectl logs configmap-mount-dir-pod
```

#### **Expected Output:**
```
value1
value2
```

The contents of the `key1.txt` and `key2.txt` files are displayed, showing that the ConfigMap has been mounted correctly.

[Back to TOC](#table-of-contents)

---

#### **Option 2: Using Specific Keys as Environment Variables**

You can also reference individual keys from the ConfigMap as environment variables.

##### **Pod YAML File: `pod-env-dir.yaml`**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-env-dir-pod
spec:
  containers:
  - name: my-container
    image: busybox
    command: [ "sh", "-c", "env" ]
    env:
    - name: KEY1
      valueFrom:
        configMapKeyRef:
          name: my-config
          key: key1.txt
    - name: KEY2
      valueFrom:
        configMapKeyRef:
          name: my-config
          key: key2.txt
  restartPolicy: Never
```

In this example, the `key1.txt` and `key2.txt` keys from the ConfigMap are set as environment variables in the container.

##### **Command to Apply Pod YAML:**
```bash
kubectl apply -f pod-env-dir.yaml
```

#### **Check the Logs to Verify:**
To check the environment variables, view the logs:

```bash
kubectl logs configmap-env-dir-pod
```

#### **Expected Output:**
```
KEY1=value1
KEY2=value2
```

This shows that the values from the ConfigMap have been successfully passed as environment variables.

[Back to TOC](#table-of-contents)

---

### **4. Summary**

- **Creating ConfigMap from Directory:** This method allows you to manage multiple configuration files in one ConfigMap, where each file in the directory becomes a key, and its content is the value.
- **Consuming the ConfigMap:** You can either mount the ConfigMap as files inside a container or reference individual keys as environment variables.

Using ConfigMaps from directories provides flexibility in managing configuration, particularly when dealing with multiple files.

[Back to TOC](#table-of-contents)

---

Would you like me to add more examples or screenshots to this tutorial?
