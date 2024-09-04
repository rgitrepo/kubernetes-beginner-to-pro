# ConfigMaps

---

## **Table of Contents**

1. [Introduction to ConfigMaps](#introduction)
2. [Application Architectures and ConfigMaps](#application-architectures)
3. [Understanding API Keys and Tokens](#api-keys-tokens)
4. [Security Risks with API Keys](#security-risks)
5. [Why ConfigMaps?](#why-configmaps)
6. [ConfigMaps vs Secrets](#configmaps-vs-secrets)
7. [Creating and Using ConfigMaps](#creating-using-configmaps)
8. [Mutable vs Immutable ConfigMaps](#mutable-immutable)
9. [Real-world Example: Using ConfigMaps](#real-world-example)
10. [Best Practices](#best-practices)
11. [Advanced Configuration: Using ConfigMaps in Containers](#advanced-configuration)
12. [Handling Updates in ConfigMaps](#handling-updates)
13. [Security Considerations](#security-considerations)
14. [Industry Practices for ConfigMaps](#industry-practices)
15. [Consuming ConfigMap Values in a Pod Command](#consuming-configmap-values)


---

### **1. Introduction to ConfigMaps** <a name="introduction"></a>

ConfigMaps in Kubernetes are used to store non-confidential key-value pairs. These key-value pairs can be used by applications to configure settings without hardcoding them into the application code. ConfigMaps help in decoupling configuration artifacts from image content to keep containerized applications portable.

[Back to TOC](#table-of-contents)

---

### **2. Application Architectures and ConfigMaps** <a name="application-architectures"></a>

Applications generally follow a multi-tier architecture:

- **2-tier:** Typically consists of a frontend and a backend.
- **3-tier:** Includes frontend, backend, and a database.

Additional components like caching layers and queues can also be added. ConfigMaps are particularly useful in these architectures for managing environment variables, image configurations, and other non-sensitive data.

**Example:**
- A 3-tier application might use a ConfigMap to store environment variables for the backend service.

[Back to TOC](#table-of-contents)

---

### **3. Understanding API Keys and Tokens** <a name="api-keys-tokens"></a>

When dealing with APIs, especially in the context of frontend-backend communication, API keys or tokens are often used to authenticate requests. These keys are essential but should be handled with care due to the potential security risks if exposed.

**Example:**
- A weather service API might require an API key to access data. This key could be stored in a ConfigMap for use by the frontend.

[Back to TOC](#table-of-contents)

---

### **4. Security Risks with API Keys** <a name="security-risks"></a>

Accidentally pushing API keys or tokens to version control systems like GitHub is a common security risk. Hackers often scan public repositories to find exposed keys. If a key is exposed, it can lead to unauthorized access and potentially costly consequences.

**Key Points:**
- API keys should never be hardcoded in the application code.
- Always use environment variables or ConfigMaps to manage keys.

**Output Example:**
```bash
# Example of warning in GitHub if a key is exposed
Warning: API key exposed in your repository. Please remove it immediately.
```

[Back to TOC](#table-of-contents)

---

### **5. Why ConfigMaps?** <a name="why-configmaps"></a>

ConfigMaps came into existence to address the need for storing non-confidential configuration data in Kubernetes. They are designed to handle data that is not sensitive, unlike Secrets, which are meant for sensitive information like passwords and API keys.

**Use Cases:**
1. Passing environment variables to applications.
2. Storing image configurations or other non-sensitive settings.

[Back to TOC](#table-of-contents)

---

### **6. ConfigMaps vs Secrets** <a name="configmaps-vs-secrets"></a>

While ConfigMaps store non-confidential data, Kubernetes Secrets are used for storing sensitive information like passwords, tokens, and API keys. The primary difference lies in the level of confidentiality and encryption provided by Secrets.

**Example Comparison:**
- ConfigMap: Stores non-confidential data like image URLs or environment settings.
- Secret: Stores confidential data like database passwords.

[Back to TOC](#table-of-contents)

---

### **7. Creating and Using ConfigMaps** <a name="creating-using-configmaps"></a>

ConfigMaps in Kubernetes allow you to decouple configuration data from application code, making your applications more flexible and easier to manage. This section provides detailed steps on how to create and configure ConfigMaps, with multiple examples to illustrate different use cases.

#### **Step 1: Creating a ConfigMap**

ConfigMaps can be created in several ways:

1. **From a Literal Value:**
   - You can create a ConfigMap from a literal key-value pair directly from the command line.
   
   **Command:**
   ```bash
   kubectl create configmap my-config --from-literal=key1=value1 --from-literal=key2=value2
   ```

   **Description:**
   - This command creates a ConfigMap named `my-config` with two key-value pairs: `key1=value1` and `key2=value2`.

2. **From a File:**
   - You can create a ConfigMap from a file that contains key-value pairs.
   
   **File: `config-file.properties`**
   ```properties
   key1=value1
   key2=value2
   ```
   
   **Command:**
   ```bash
   kubectl create configmap my-config --from-file=config-file.properties
   ```

   **Description:**
   - This command creates a ConfigMap named `my-config` where the content of `config-file.properties` is loaded as key-value pairs.

3. **From a Directory:**
   - You can create a ConfigMap from a directory where each file in the directory is a key, and the content of each file is the value.
   
   **Directory Structure:**
   ```
   config-dir/
   ├── key1
   └── key2
   ```
   
   **Command:**
   ```bash
   kubectl create configmap my-config --from-file=config-dir/
   ```

   **Description:**
   - This command creates a ConfigMap named `my-config`, where `key1` and `key2` are the keys, and the content of the files `key1` and `key2` are the corresponding values.

4. **From YAML Files:**
   - You can define a ConfigMap directly in a YAML file and apply it to the cluster.

   **YAML File: `my-config.yaml`**
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

   **Description:**
   - This creates a ConfigMap named `my-config` with `key1=value1` and `key2=value2` defined in the YAML file.

#### **Step 2: Configuring ConfigMaps for Use in a Pod**

After creating a ConfigMap, you can use it in your Pods by referencing it in your Pod definitions. Below are different examples of how to configure and use ConfigMaps.

**Example 1: Using ConfigMap as Environment Variables**

**YAML File: `pod-env.yaml`**
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

**Steps to Configure:**
1. **Create the ConfigMap:**
   - Use the earlier `my-config.yaml` file or create it using the command:
   ```bash
   kubectl apply -f my-config.yaml
   ```

2. **Create the Pod:**
   - Apply the `pod-env.yaml` file:
   ```bash
   kubectl apply -f pod-env.yaml
   ```

3. **Check Pod Logs:**
   - Verify that the environment variables are correctly set by checking the logs:
   ```bash
   kubectl logs env-pod
   ```

   **Expected Output:**
   ```
   ENV_KEY1=value1
   ENV_KEY2=value2
   ```

**Example 2: Using ConfigMap as a Volume**

**YAML File: `pod-volume.yaml`**
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
      readOnly: true
    command: ["/bin/sh", "-c", "ls -l /etc/config && cat /etc/config/key1 && cat /etc/config/key2"]
  volumes:
  - name: config-volume
    configMap:
      name: my-config
```

**Steps to Configure:**
1. **Create the ConfigMap:**
   - Ensure the ConfigMap `my-config` is created using the `my-config.yaml` file:
   ```bash
   kubectl apply -f my-config.yaml
   ```

2. **Create the Pod:**
   - Apply the `pod-volume.yaml` file:
   ```bash
   kubectl apply -f pod-volume.yaml
   ```

3. **Check Pod Logs:**
   - Verify the files are correctly mounted and their content is accurate:
   ```bash
   kubectl logs volume-pod
   ```

   **Expected Output:**
   ```
   total 8
   -r--r--r-- 1 root root 7 Jul 15 12:00 key1
   -r--r--r-- 1 root root 7 Jul 15 12:00 key2
   value1
   value2
   ```

**Example 3: ConfigMap with Complex Data**

Sometimes, you may want to store complex data structures, like entire configuration files, in a ConfigMap.

**YAML File: `complex-config.yaml`**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: complex-config
data:
  app.conf: |
    [app]
    setting1=value1
    setting2=value2
  db.conf: |
    [database]
    host=db.example.com
    port=5432
    user=admin
    password=secret
```

**Steps to Configure:**
1. **Create the ConfigMap:**
   - Apply the `complex-config.yaml` file:
   ```bash
   kubectl apply -f complex-config.yaml
   ```

2. **Create a Pod to Use the ConfigMap:**

**YAML File: `pod-complex.yaml`**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: complex-pod
spec:
  containers:
  - name: complex-container
    image: busybox
    volumeMounts:
    - name: complex-volume
      mountPath: "/etc/config"
      readOnly: true
    command: ["/bin/sh", "-c", "cat /etc/config/app.conf && cat /etc/config/db.conf"]
  volumes:
  - name: complex-volume
    configMap:
      name: complex-config
```

3. **Apply the Pod Definition:**
   - Create the Pod using the command:
   ```bash
   kubectl apply -f pod-complex.yaml
   ```

4. **Check the Pod Logs:**
   - Inspect the output to verify the configuration files were correctly mounted and their contents are accurate:
   ```bash
   kubectl logs complex-pod
   ```

   **Expected Output:**
   ```
   [app]
   setting1=value1
   setting2=value2
   [database]
   host=db.example.com
   port=5432
   user=admin
   password=secret
   ```

---

**Summary of Steps:**

1. **Create ConfigMap:**
   - Use literal values, files, directories, or YAML files.
   - Apply them using `kubectl apply -f <filename>.yaml` or `kubectl create configmap <name> --from-file=<file>`.

2. **Configure Pod to Use ConfigMap:**
   - Use the `env` field to pass ConfigMap values as environment variables.
   - Use the `volumes` and `volumeMounts` fields to mount ConfigMap data as files within the container.

3. **Apply Pod Configuration:**
   - Deploy the Pod using `kubectl apply -f <pod-filename>.yaml`.

4. **Verify Configuration:**
   - Check the logs or inspect the filesystem of the running container to ensure the ConfigMap values are correctly consumed.

By following these detailed steps and examples, you should be able to effectively create, configure, and use ConfigMaps in your Kubernetes applications, enhancing both flexibility and security in your deployments.

[Back to TOC](#table-of-contents)


---

### **8. Mutable vs Immutable ConfigMaps** <a name="mutable-immutable"></a>

ConfigMaps are mutable by default, meaning their values can be changed after they are created. However, in industry practices, making a ConfigMap immutable is often preferred, as it prevents accidental changes that could lead to application failures.

**Making ConfigMap Immutable:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-config
  annotations:
    "kubectl.kubernetes.io/last-applied-configuration": "true"
    "kubectl.kubernetes.io/immutable": "true"
data:
  key: "value"
```

**Why Make ConfigMaps Immutable?**
- **Consistency:** Ensures that the configuration does not change unexpectedly during the application’s lifecycle.
- **Security:** Prevents unauthorized or accidental modifications to the configuration.
- **Best Practice:** Immutability is a best practice in the industry, especially in environments where stability and predictability are crucial.

[Back to TOC](#table-of-contents)

---

### **9. Real-world Example: Using ConfigMaps** <a name="real-world-example"></a>

Consider a 3-tier application where the backend service needs to connect to a database. The database URL and logging level can be stored in a ConfigMap, allowing the backend service to fetch this configuration without hardcoding it.

**YAML Example:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: backend-config
data:
  database_url: "jdbc:mysql://db:3306/mydb"
  log_level: "DEBUG"
```

**Deploying the Application:**
- The backend service retrieves the `database_url` and `log_level` from the ConfigMap during startup.

[Back to TOC](#table-of-contents)

---

### **10. Best Practices** <a name="best-practices"></a>

- **Decouple Configuration from Code:** Always use ConfigMaps to manage non-sensitive configuration data, keeping it separate from the application code.
- **Use Secrets for Sensitive Data:** Store sensitive information in Kubernetes Secrets, not in ConfigMaps.
- **Immutable ConfigMaps:** Use immutable ConfigMaps for critical configurations to prevent accidental changes.
- **Monitor for Exposed Keys:** Regularly check your version control systems to ensure that no sensitive keys have been accidentally pushed.

**Example of Monitoring Command:**
```bash
# Using a tool like TruffleHog to scan for exposed keys
trufflehog --regex --entropy=True --max_depth=50 https://github.com/your-repo.git
```

[Back to TOC](#table-of-contents)

---

### **11. Advanced Configuration: Using ConfigMaps in Containers** <a name="advanced-configuration"></a>

ConfigMaps can be consumed in various ways within Kubernetes Pods, with one of the most common methods being the use of volumes. Volumes provide a way to share data between containers in a Pod or to persist data across container restarts. When using ConfigMaps, mounting them as volumes is a powerful and flexible way to inject configuration data into your containers as files. This method allows your application to read configuration files from the filesystem, which can be more straightforward than using environment variables for some applications.

#### **Mounting ConfigMaps as Volumes**

When you mount a ConfigMap as a volume in a Pod, the keys in the ConfigMap become file names, and the corresponding values become the file content. This allows your application to read the configuration data directly from the mounted directory.

**Example:**

Let’s say you have a ConfigMap that contains two configuration keys, `app-config` and `db-config`. You can mount this ConfigMap as a volume in your Pod as follows:

**ConfigMap YAML:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  app-config: |
    setting1=value1
    setting2=value2
  db-config: |
    database=prod-db
    user=admin
    password=secret
```

**Pod YAML File:**
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
      readOnly: true
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```

**Explanation:**

1. **ConfigMap Definition:**
   - The ConfigMap `app-config` contains two keys: `app-config` and `db-config`. Each key stores a multi-line string that could represent a configuration file.

2. **Mounting as a Volume:**
   - In the Pod specification, the `app-config` ConfigMap is mounted as a volume named `config-volume`.
   - The `volumeMounts` section specifies that this volume will be mounted at the path `/etc/config` in the container's filesystem.

3. **File Structure in the Container:**
   - Once mounted, the container will have two files available at `/etc/config/`:
     - `/etc/config/app-config`: Contains the content defined under the `app-config` key in the ConfigMap.
     - `/etc/config/db-config`: Contains the content defined under the `db-config` key in the ConfigMap.
   - Your application can read these files as it would read any other file on the filesystem.

4. **Read-Only Volumes:**
   - The `readOnly: true` flag is used when mounting the volume. This ensures that the files from the ConfigMap are mounted as read-only, meaning that they cannot be modified by the container.
   - **Importance of `readOnly`:** 
     - **Security:** Prevents unauthorized or accidental modifications to the configuration files, which could lead to application failures or security vulnerabilities.
     - **Consistency:** Ensures that the configuration remains unchanged during the runtime of the application, providing consistency across multiple instances or restarts of the container.

#### **Why Use ConfigMaps as Volumes?**

- **Dynamic Configuration:** ConfigMaps allow you to externalize configuration from your application code, enabling you to change configuration without altering the application image.
- **File-Based Configuration:** Some applications are designed to read configurations from files rather than environment variables. Mounting a ConfigMap as a volume makes it easier to supply these configurations.
- **Automatic Updates:** When a ConfigMap is mounted as a volume, Kubernetes automatically checks for updates and refreshes the content of the files in the volume. This means that your application can pick up configuration changes without needing to restart the Pod, provided the application can handle dynamic configuration reloading.

#### **Best Practices When Using ConfigMaps as Volumes:**

1. **Use Read-Only Mounts:**
   - Always set `readOnly: true` when mounting ConfigMaps as volumes. This prevents any accidental changes to the configuration files and maintains the integrity of your application's configuration.

2. **Organize Configuration Files:**
   - Use meaningful key names in your ConfigMap that reflect the purpose of the configuration. This makes it easier to understand the configuration structure within the container’s filesystem.

3. **Limit Access:** 
   - Control which containers or services can access the ConfigMap to maintain security. Only mount the ConfigMap to containers that require the configuration.

4. **Monitor Changes:**
   - Be aware that mounting ConfigMaps as volumes allows for dynamic updates. Ensure that your application is capable of handling changes to the configuration files during runtime, or make sure to manage the timing of ConfigMap updates carefully.

**Example: Monitoring and Logging Applications:**
- Consider a logging application that reads its configuration from a file. By using a ConfigMap mounted as a volume, you can update the logging configuration dynamically, and the application can adapt without requiring a restart.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: logging-pod
spec:
  containers:
  - name: logging-container
    image: logging-app
    volumeMounts:
    - name: logging-config
      mountPath: "/etc/logging"
      readOnly: true
  volumes:
  - name: logging-config
    configMap:
      name: logging-configmap
```

In this example, the logging application can read its configuration from `/etc/logging/logging-config`, and you can update the logging levels or formats dynamically by editing the ConfigMap without interrupting the application’s service.


[Back to TOC](#table-of-contents)

---

### **12. Handling Updates in ConfigMaps** <a name="handling-updates"></a>

When a ConfigMap is updated, the changes do not automatically propagate to the running Pods unless the ConfigMap is mounted as a volume. If you are using ConfigMaps as **environment variables**, you will need to **restart** the Pod to apply the changes.

**Auto-Reloading ConfigMaps with Volumes:**
- When ConfigMaps are mounted as **volumes**, the updated values will be **automatically** loaded into the Pod without needing a restart. Kubernetes periodically checks for changes and updates the volume contents.

**Command to Update ConfigMap:**
```bash
kubectl edit configmap backend-config
```

**Restarting Pods (if using environment variables):**
```bash
kubectl rollout restart deployment my-deployment
```

[Back to TOC](#table-of-contents)

---

### **13. Security Considerations** <a name="security-considerations"></a>

To enhance the security of ConfigMaps, especially when they are mounted as volumes, consider the following best practices:

- **Read-Only Volumes:** Always mount ConfigMaps as read-only to prevent any unauthorized write operations.
- **Access Control:** Limit access to ConfigMaps through Kubernetes RBAC (Role-Based Access Control) to ensure that only authorized users and services can access or modify them.

**Example:**
```yaml
volumeMounts:
- name: config-volume
  mountPath: "/etc/config"
  readOnly: true
```

**Additional Security Considerations:**
- Be cautious about storing sensitive data in ConfigMaps. If there is any doubt, use Secrets instead.
- Regularly audit ConfigMaps and their usage within your cluster to ensure that they adhere to security policies.

[Back to TOC](#table-of-contents)

---

### **14. Industry Practices for ConfigMaps** <a name="industry-practices"></a>

In the industry, there are several practices that are considered best when working with ConfigMaps:

#### **1. Naming Conventions:**
- **Purposeful Naming:** Ensure that the name of the ConfigMap reflects its purpose. This makes it easier for team members and future developers to understand what the ConfigMap is for without confusion. For example, if the ConfigMap is storing database-related configurations, name it something like `db-config`.

#### **2. Immutability:**
- **Immutability as a Default:** As mentioned earlier, making ConfigMaps immutable is a common industry practice. This ensures that once a ConfigMap is set, it cannot be modified, reducing the risk of accidental or unauthorized changes. This practice is particularly important in production environments where stability and predictability are paramount.

#### **3. Controlled Access:**
- **RBAC Policies:** Use Kubernetes Role-Based Access Control (RBAC) to control who can create, update, and delete ConfigMaps. This helps in maintaining the integrity of your configuration data.

#### **4. Consistent Formatting:**
- **Key-Value Pairs:** Ensure that the keys in your ConfigMaps are consistently named and follow a specific format or convention. This consistency helps in avoiding errors when these values are referenced in your application code or deployment manifests.

#### **5. Environment-Specific Configurations:**
- **Separate ConfigMaps per Environment:** For applications that run in multiple environments (e.g., development, staging, production), it's a good practice to have separate ConfigMaps for each environment. This allows you to tailor configurations without affecting other environments.

#### **6. Avoid Sensitive Data in ConfigMaps:**
- **Secrets for Sensitive Data:** Always store sensitive information such as API keys, database credentials, and tokens in Kubernetes Secrets, not in ConfigMaps. ConfigMaps are not encrypted by default, so they should only be used for non-sensitive configuration data.

**Example of a Well-Named ConfigMap:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: frontend-app-config
data:
  api_base_url: "https://api.example.com"
  environment: "production"
```

**Ensuring Read-Only ConfigMaps:**
- Always mount ConfigMaps as read-only in your Pods to prevent any potential write operations that could modify your configuration data unexpectedly.

```yaml
volumeMounts:
- name: config-volume
  mountPath: "/etc/config"
  readOnly: true
```

By following these industry practices, you can ensure that your use of ConfigMaps in Kubernetes is secure, reliable, and maintainable.

[Back to TOC](#table-of-contents)

---

### **15. Consuming ConfigMap Values in a Pod Command** <a name="consuming-configmap-values"></a>

In Kubernetes, you can pass values from a ConfigMap as environment variables to a container within a Pod. These environment variables can then be used in commands executed by the container. This is a common pattern when you want to configure the behavior of your application dynamically without hardcoding values into your container image.

#### **Example Scenario:**

Let’s assume you have a ConfigMap that stores two configuration keys, `SPECIAL_LEVEL_KEY` and `SPECIAL_TYPE_KEY`, and you want to use these values in a command executed by the container when it starts.

**ConfigMap YAML:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
data:
  SPECIAL_LEVEL_KEY: "Hard"
  SPECIAL_TYPE_KEY: "Boss"
```

**Pod YAML File:**
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

#### **Explanation of What’s Going On:**

1. **ConfigMap Definition:**
   - The ConfigMap `special-config` contains two key-value pairs:
     - `SPECIAL_LEVEL_KEY` is set to `"Hard"`.
     - `SPECIAL_TYPE_KEY` is set to `"Boss"`.

2. **Pod Specification:**
   - The Pod `special-pod` is defined with a single container named `special-container`, using the `busybox` image.
   - The `command` field specifies the command that the container will run when it starts. Here, the command is `"/bin/echo"`, which will print the values passed to it.
   - The values to be printed are `$(SPECIAL_LEVEL_KEY) $(SPECIAL_TYPE_KEY)`. These are placeholders that will be replaced by the values of the corresponding environment variables when the command is executed.

3. **Environment Variables:**
   - The environment variables `SPECIAL_LEVEL_KEY` and `SPECIAL_TYPE_KEY` are defined in the `env` section of the container specification.
   - Each environment variable is populated with the value from the `special-config` ConfigMap using the `valueFrom` field. Specifically:
     - `SPECIAL_LEVEL_KEY` is sourced from the `SPECIAL_LEVEL_KEY` key in the `special-config` ConfigMap.
     - `SPECIAL_TYPE_KEY` is sourced from the `SPECIAL_TYPE_KEY` key in the `special-config` ConfigMap.

4. **Command Execution:**
   - When the container starts, the `"/bin/echo"` command is executed. The environment variables `SPECIAL_LEVEL_KEY` and `SPECIAL_TYPE_KEY` are substituted with their respective values from the ConfigMap.
   - The final command that gets executed is:
     ```bash
     /bin/echo "Hard Boss"
     ```
   - This command will output `"Hard Boss"` to the container’s standard output.

#### **Learning Points:**

- **Dynamic Configuration:** By using ConfigMaps and environment variables, you can dynamically configure containers at runtime without modifying the container image. This makes your deployments more flexible and easier to manage.
  
- **Environment Variable Substitution:** The use of `$(...)` in the `command` field is a shell feature that allows environment variable substitution. Kubernetes replaces these placeholders with the actual values of the environment variables at runtime.

- **ConfigMap Consumption:** The `valueFrom.configMapKeyRef` field is crucial for pulling specific values from a ConfigMap and using them as environment variables in your containers.

- **Command Execution:** The command field in the Pod specification allows you to override the default command in the container image with a custom command, making it possible to tailor the container's behavior based on dynamic input from ConfigMaps.

**Best Practice:**
- Always validate that the environment variables are correctly set and that the ConfigMap keys exist to avoid runtime errors. 

[Back to TOC](#table-of-contents)

