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
   - [Example: Adding CPU and Memory to Pods](#example-adding-cpu-and-memory-to-pods)
   - [Validating Policies](#validating-policies)
   - [Viewing Policy Reports](#viewing-policy-reports)
6. [Advanced Policy Creation](#advanced-policy-creation)
   - [Runtime Class Enforcement](#runtime-class-enforcement)
7. [Additional Resources and Self-Study](#additional-resources-and-self-study)
8. [Conclusion](#conclusion)

---

### 1. Introduction
In this tutorial, we will dive deep into Kubernetes admission controllers, specifically focusing on the Kyverno tool. Kyverno is a Kubernetes-native policy engine that simplifies the management of policies and security configurations. This guide will walk you through the setup, creation, and application of Kyverno policies, along with examples to solidify your understanding.

[Back to TOC](#table-of-contents)

---

### 2. What is Kyverno?
Kyverno is a policy engine designed for Kubernetes. It allows you to manage and enforce policies for your Kubernetes resources using YAML configurations. Being Kubernetes-native, Kyverno does not require you to learn a new language or complex configurations, making it a beginner-friendly yet powerful tool.

[Back to TOC](#table-of-contents)

---

### 3. Understanding Kubernetes Admission Controllers
Kubernetes admission controllers are key components that intercept requests to the Kubernetes API server before they are persisted as objects in etcd. They help enforce policies and ensure security within the cluster.

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

#### Example: Adding CPU and Memory to Pods
Let's start with a simple Kyverno policy that adds default CPU and memory specifications to any pod that doesn’t already have them defined.

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

#### Validating Policies
Kyverno allows you to validate policies to ensure they meet certain conditions. For example, you can enforce that a pod must have a specific label or runtime class.

1. **Create a Validation Policy:**
   ```yaml
   apiVersion: kyverno.io/v1
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
