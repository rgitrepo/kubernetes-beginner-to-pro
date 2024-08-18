### Kubernetes Admission Controllers: Kyverno Tutorial

#### Table of Contents
1. [Introduction](#introduction)
2. [What is Kyverno?](#what-is-kyverno)
3. [Understanding Kubernetes Admission Controllers](#understanding-kubernetes-admission-controllers)
   - [Authentication](#authentication)
   - [Authorization](#authorization)
   - [Admission Control](#admission-control)
4. [Setting Up Kyverno](#setting-up-kyverno)
   - [Installing Kyverno Using Helm](#installing-kyverno-using-helm)
   - [Validating Kyverno Installation](#validating-kyverno-installation)
5. [Creating and Applying Kyverno Policies](#creating-and-applying-kyverno-policies)
   - [Use Case: Adding CPU and Memory to Pods](#use-case-adding-cpu-and-memory-to-pods)
   - [Use Case: Enforcing Specific Images](#use-case-enforcing-specific-images)
   - [Use Case: Adding Specific Labels](#use-case-adding-specific-labels)
   - [Validating Policies](#validating-policies)
   - [Viewing Policy Reports](#viewing-policy-reports)
6. [Advanced Policy Creation](#advanced-policy-creation)
   - [Runtime Class Enforcement](#runtime-class-enforcement)
7. [Additional Resources and Self-Study](#additional-resources-and-self-study)
8. [Conclusion](#conclusion)

---

### 1. Introduction
In this tutorial, we will explore Kubernetes admission controllers, focusing specifically on the Kyverno tool. Kyverno is a Kubernetes-native policy engine that simplifies policy management and security configurations. This guide will walk you through the setup, creation, and application of Kyverno policies, including several real-world use cases to help you apply what you learn.

[Back to TOC](#table-of-contents)

---

### 2. What is Kyverno?
Kyverno is a policy engine designed specifically for Kubernetes. It enables you to manage and enforce policies on your Kubernetes resources using YAML configurations. As a Kubernetes-native tool, Kyverno does not require you to learn a new programming language. Instead, you can leverage the power of YAML to define and enforce your policies. Kyverno’s official documentation contains 200-225 pre-defined policy examples, making it easy to get started. You can simply copy and paste these policies, and writing new ones is straightforward as well.

[Back to TOC](#table-of-contents)

---

### 3. Understanding Kubernetes Admission Controllers
Kubernetes admission controllers are essential components that intercept requests to the Kubernetes API server before they are persisted as objects in etcd. They enforce policies and ensure security within the cluster.

#### Authentication
When you issue a `kubectl` command, the request first goes to the API server, which checks if you are authenticated. This step verifies if you have the authority to log in.

#### Authorization
Once authenticated, the API server checks if you have the necessary permissions to make the requested changes.

#### Admission Control
Finally, the admission control phase enforces specific policies, such as requiring labels or annotations on resources, validating resource configurations, or mutating them to include additional specifications like CPU and memory.

[Back to TOC](#table-of-contents)

---

### 4. Setting Up Kyverno

#### Installing Kyverno Using Helm
To install Kyverno, you'll need Helm, a package manager for Kubernetes. Here’s how to install Kyverno using Helm:

1. **Add the Helm Repository:**
   ```bash
   helm repo add kyverno https://kyverno.github.io/kyverno/
   helm repo update
   ```
   This command adds the Kyverno Helm repository and updates your local Helm repo cache.

2. **Install Kyverno:**
   ```bash
   helm install kyverno kyverno/kyverno --namespace kyverno --create-namespace
   ```
   This command installs Kyverno in the `kyverno` namespace.

#### Validating Kyverno Installation
To validate the installation, you can check the status of the Kyverno pods using the following command:
```bash
kubectl get pods -n kyverno
```
You should see the Kyverno pods running, indicating a successful installation.

[Back to TOC](#table-of-contents)

---

### 5. Creating and Applying Kyverno Policies

#### Use Case: Adding CPU and Memory to Pods
One common use case for Kyverno is to ensure that every pod in your cluster has specified CPU and memory limits. This can help prevent resource hogging and ensure fair distribution of resources across your cluster.

1. **Create a Policy YAML File:**
   ```yaml
   apiVersion: kyverno.io/v1
   kind: ClusterPolicy
   metadata:
     name: add-resources
   spec:
     rules:
       - name: add-cpu-and-memory
         match:
           resources:
             kinds:
             - Pod
         mutate:
           patchStrategicMerge:
             spec:
               containers:
                 - (name): "*"
                   resources:
                     requests:
                       memory: "100Mi"
                       cpu: "100m"
   ```
   - **Explanation:**
     - `match`: This key specifies the types of resources that the policy should apply to. In this case, it targets `Pod` resources.
     - `mutate`: This block defines how the matched resources should be modified. Here, it adds default CPU and memory requests to all containers within the pods.
     - `patchStrategicMerge`: This is the method used by Kyverno to apply changes. It merges the specified fields with the existing configuration of the resource.

2. **Apply the Policy:**
   ```bash
   kubectl apply -f add-resources.yaml
   ```
   This command applies the policy to your cluster.

3. **Create a Pod Without Resource Requests:**
   ```bash
   kubectl run nginx --image=nginx -- sleep 1d
   ```

4. **Check the Applied Policy:**
   Describe the pod to verify that the CPU and memory were automatically added:
   ```bash
   kubectl describe pod nginx
   ```

   You should see the following under the pod's spec:
   ```yaml
   Requests:
     cpu:  100m
     memory:  100Mi
   ```

[Back to TOC](#table-of-contents)

---

#### Use Case: Enforcing Specific Images
Another practical application of Kyverno is to enforce the use of specific container images in your Kubernetes pods. This can be crucial for security and compliance, ensuring that only approved images are used across your cluster.

1. **Create a Policy YAML File:**
   ```yaml
   apiVersion: kyverno.io/v1
   kind: ClusterPolicy
   metadata:
     name: enforce-image
   spec:
     rules:
       - name: require-specific-image
         match:
           resources:
             kinds:
             - Pod
         validate:
           message: "Only nginx image is allowed."
           pattern:
             spec:
               containers:
                 - (name): "*"
                   image: "nginx"
   ```
   - **Explanation:**
     - `validate`: This block is used to enforce specific conditions on the resource. In this case, it ensures that all containers in the pod use the `nginx` image.
     - `message`: The message field specifies the error message that will be returned if the validation fails.
     - `pattern`: This key defines the expected pattern that the resource must match. If the resource doesn't match this pattern, it will be rejected.

2. **Apply the Policy:**
   ```bash
   kubectl apply -f enforce-image.yaml
   ```

3. **Test the Policy:**
   Try creating a pod with a different image:
   ```bash
   kubectl run test-pod --image=busybox -- sleep 1d
   ```
   You should receive an error message indicating that the pod was blocked due to the validation policy.

[Back to TOC](#table-of-contents)

---

#### Use Case: Adding Specific Labels
You may want to ensure that all pods in a certain namespace or across the entire cluster have specific labels. This can be useful for categorization, monitoring, or applying additional policies.

1. **Create a Policy YAML File:**
   ```yaml
   apiVersion: kyverno.io/v1
   kind: ClusterPolicy
   metadata:
     name: add-label
   spec:
     rules:
       - name: add-environment-label
         match:
           resources:
             kinds:
             - Pod
         mutate:
           patchStrategicMerge:
             metadata:
               labels:
                 environment: "production"
   ```
   - **Explanation:**
     - `metadata`: This block within `mutate` allows you to modify the metadata of the resource, such as adding or updating labels.
     - `labels`: Here, a specific label `environment: production` is added to all pods that match the criteria.

2. **Apply the Policy:**
   ```bash
   kubectl apply -f add-label.yaml
   ```

3. **Create a Pod Without Labels:**
   ```bash
   kubectl run nginx --image=nginx -- sleep 1d
   ```

4. **Check the Applied Policy:**
   Describe the pod to verify that the label was automatically added:
   ```bash
   kubectl describe pod nginx
   ```

   You should see the following under the pod's metadata:
   ```yaml
   Labels:
     environment: production
   ```

[Back to TOC](#table-of-contents)

---

#### Validating Policies
Kyverno allows you to validate policies to ensure they meet certain conditions. For example, you can enforce that a pod must have a specific label or runtime class.

1. **Create a Validation Policy:**
   ```yaml
   apiVersion: ky

verno.io/v1
   kind: ClusterPolicy
   metadata:
     name: validate-runtime-class
   spec:
     validationFailureAction: enforce
     rules:
       - name: check-runtime-class
         match:
           resources:
             kinds:
             - Pod
         validate:
           message: "Runtime class must be 'kata'."
           pattern:
             spec:
               runtimeClassName: "kata"
   ```
   - **Explanation:**
     - `validationFailureAction`: This key determines the action Kyverno should take when a policy fails. In this case, `enforce` will block the resource creation if the policy is not met. Another common value is `audit`, which logs the failure without blocking the resource.
     - `runtimeClassName`: This specifies the required runtime class for the pod. If a pod does not match this runtime class, it will be rejected.

2. **Apply the Validation Policy:**
   ```bash
   kubectl apply -f validate-runtime-class.yaml
   ```

3. **Test the Policy:**
   Try creating a pod without the required runtime class:
   ```bash
   kubectl run test-pod --image=nginx -- sleep 1d
   ```
   You should receive an error message indicating that the pod was blocked due to the validation policy.

[Back to TOC](#table-of-contents)

---

#### Viewing Policy Reports
Kyverno generates policy reports that help you understand which resources have passed or failed policy checks.

1. **View Policy Reports:**
   ```bash
   kubectl get policyreports
   ```
   This command lists the policy reports, showing you which policies passed or failed.

[Back to TOC](#table-of-contents)

---

### 6. Advanced Policy Creation

#### Runtime Class Enforcement
Kyverno allows you to enforce the use of specific runtime classes, such as `kata`, across your cluster. This is especially useful for ensuring that all pods conform to security or performance standards.

1. **Enforce Runtime Class:**
   The validation policy we created earlier (`validate-runtime-class.yaml`) ensures that only pods with the `kata` runtime class can be created.

2. **Testing and Validation:**
   When you attempt to create a pod without the `kata` runtime class, it will be denied, helping maintain cluster security.

[Back to TOC](#table-of-contents)

---

### 7. Additional Resources and Self-Study
To further enhance your understanding of Kyverno and Kubernetes policies, consider exploring the following resources:
- Kyverno [official documentation](https://kyverno.io/)
- Kubernetes [Admission Controllers](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)
- Helm [official documentation](https://helm.sh/docs/)

Self-study is crucial to mastering these tools, and I recommend experimenting with different policies and configurations in a test environment.

[Back to TOC](#table-of-contents)

---

### 8. Conclusion
Kyverno is a powerful yet user-friendly tool for managing Kubernetes policies. By following this tutorial, you should now have a solid understanding of how to set up Kyverno, create and apply policies, and validate your Kubernetes resources. Continue exploring and experimenting to deepen your knowledge and ensure the security and efficiency of your Kubernetes clusters.

[Back to TOC](#table-of-contents)
