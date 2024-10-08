

## **Network Policies in Kubernetes**

### **Updated Table of Contents**

1. [Introduction to Network Policies](#introduction-to-network-policies)
2. [Ingress and Egress Traffic Explained](#ingress-and-egress-traffic-explained)
3. [Understanding Network Policies in Kubernetes](#understanding-network-policies-in-kubernetes)
4. [How Network Policies Are Applied](#how-network-policies-are-applied)
    - [Interview Question](#what-happens-if-no-network-policy-is-applied-to-a-pod)
5. [Examples of Network Policies](#examples-of-network-policies)
    - [Example 1: Denying All Traffic by Default](#denying-all-traffic-by-default)
        - [Interview Question](#what-happens-if-no-network-policy-is-applied-to-a-pod)
    - [Example 2: Allowing Traffic Only Between Specific Pods](#allowing-traffic-only-between-specific-pods)
        - [Interview Question](#how-can-you-restrict-pod-to-pod-communication-using-network-policies)
    - [Example 3: Restricting Traffic by CIDR Block](#restricting-traffic-by-cidr-block)
6. [Working with Labels and Selectors](#working-with-labels-and-selectors)
    - [Example of Confusion with Multiple PodSelectors](#example-of-confusion-with-multiple-podselectors)
        - [Interview Question](#what-is-the-difference-between-the-podselector-at-the-top-level-of-a-network-policy-and-the-one-used-inside-the-ingress-egress-rules)
7. [Using Namespace Selectors](#using-namespace-selectors)
8. [Working with IP Blocks](#working-with-ip-blocks)
9. [Practical Usage in Cloud and Common Issues](#practical-usage-in-cloud-and-common-issues)
10. [Conclusion](#conclusion)

---

### **1. Introduction to Network Policies** <a name="introduction-to-network-policies"></a>

**Network Policies** in Kubernetes are used to control the flow of network traffic to and from Pods within a cluster. They act like firewall rules for Pods, restricting the ingress (incoming) and egress (outgoing) traffic based on certain rules and selectors.

**Key concepts**:
- **Ingress traffic**: Traffic coming into a Pod.
- **Egress traffic**: Traffic leaving a Pod.
- By default, **all traffic** (both ingress and egress) is allowed in Kubernetes until restricted by a network policy.

**[Back to TOC](#updated-table-of-contents)**

---

### **2. Ingress and Egress Traffic Explained** <a name="ingress-and-egress-traffic-explained"></a>

To understand network policies, it’s important to grasp the difference between ingress and egress traffic:

- **Ingress Traffic**: Traffic that is entering the target Pod from another source (another Pod or external service).
- **Egress Traffic**: Traffic that is leaving the target Pod to another destination (another Pod or external service).

For example, imagine two services:
- **Service A** (frontend) sends requests to **Service B** (backend).
- When Service A sends traffic, it's **egress** from Service A's perspective and **ingress** from Service B's perspective.

**Real-world analogy**: Imagine playing a game of catch. When you throw the ball, it's leaving your hand (egress) and entering your friend's hands (ingress). The same principle applies to traffic between Pods.

**[Back to TOC](#updated-table-of-contents)**

---

### **3. Understanding Network Policies in Kubernetes** <a name="understanding-network-policies-in-kubernetes"></a>

Network policies in Kubernetes are applied using YAML files that define how traffic is managed for certain Pods. These policies allow users to:
1. **Restrict traffic** based on source and destination.
2. **Allow traffic** selectively between specific Pods or services.
3. Define whether the policy affects **ingress**, **egress**, or both.

### Components of a Network Policy YAML:
1. **Metadata**: The name of the policy and namespace.
2. **Pod Selector**: The set of Pods the policy applies to.
3. **Policy Types**: Whether it applies to ingress, egress, or both.
4. **Rules**: Specific conditions (from where traffic is allowed or blocked).

Here’s an example YAML skeleton of a basic Network Policy:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-some-traffic
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
```

**[Back to TOC](#updated-table-of-contents)**

---

### **4. How Network Policies Are Applied** <a name="how-network-policies-are-applied"></a>

By default, Kubernetes allows all traffic into and out of Pods. Applying a Network Policy changes this behavior. A key **interview question** often asked is whether any traffic restrictions exist by default in Kubernetes — **there are none** until a policy is applied.

- **Pod Selector**: Specifies the Pods affected by the policy.
- **Policy Types**: Specifies if the policy applies to ingress, egress, or both.
- **Rules**: Defines from where traffic can be allowed or denied.

**Important Note**: When a policy is applied, only the traffic allowed by that policy will be permitted. All other traffic will be blocked.

<a name="what-happens-if-no-network-policy-is-applied-to-a-pod"></a>**Interview Question**:  
- **What happens if no network policy is applied to a Pod?**
    - By default, all traffic (both ingress and egress) is allowed.

**[Back to TOC](#updated-table-of-contents)**

---

### **5. Examples of Network Policies** <a name="examples-of-network-policies"></a>

#### **Example 1: Denying All Traffic by Default** <a name="denying-all-traffic-by-default"></a>

This policy blocks all incoming and outgoing traffic for the selected Pods:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

**Explanation**:
- The `podSelector: {}` selects all Pods in the namespace.
- Both `Ingress` and `Egress` traffic are blocked.

To verify the applied policy and see which traffic is restricted, use the following command:

```bash
kubectl describe networkpolicy deny-all
```

**Output Example**:
```plaintext
Name:         deny-all
Namespace:    default
PodSelector:  <none> (Allowing the specific Pods to be affected)
PolicyTypes:  Ingress, Egress
Allowed Ingress: None
Allowed Egress: None
```

<a name="what-happens-if-no-network-policy-is-applied-to-a-pod"></a>**Interview Question**:  
- **What happens if no network policy is applied to a Pod?**
    - By default, all traffic (both ingress and egress) is allowed.

**[Back to TOC](#updated-table-of-contents)**

---

#### **Example 2: Allowing Traffic Only Between Specific Pods** <a name="allowing-traffic-only-between-specific-pods"></a>

This example shows how to allow traffic between specific Pods:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
```

**Explanation**:
- The policy applies to Pods labeled as `app: backend`.
- Only Pods labeled `app: frontend` are allowed to communicate with the backend Pods.

You can use the `kubectl describe networkpolicy allow-frontend-to-backend` command to check this policy:

```bash
kubectl describe networkpolicy allow-frontend-to-backend
```

**Output Example**:
```plaintext
Name:         allow-frontend-to-backend
Namespace:    default
PodSelector:  app=backend
PolicyTypes:  Ingress
Allowed Ingress: 
  - From:
    - PodSelector: app=frontend
```

<a name="how-can-you-restrict-pod-to-pod-communication-using-network-policies"></a>**Interview Question**:  
- **How can you restrict Pod-to-Pod communication using network policies?**
    - By applying specific network policies, such as the one above, where only selected Pods (like frontend) are allowed to communicate with backend Pods.

**[Back to TOC](#updated-table-of-contents)**

---

#### **Example 3: Restricting Traffic by CIDR Block** <a name="restricting-traffic-by-cidr-block"></a>

This example demonstrates how to use a CIDR block to restrict traffic from a specific range of IP addresses.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-cidr
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress


  ingress:
  - from:
    - ipBlock:
        cidr: 10.0.0.0/16
        except:
        - 10.0.1.0/24
```

**Explanation**:
- The policy applies to Pods labeled as `app: backend`.
- It allows traffic from the CIDR block `10.0.0.0/16` except for the IP range `10.0.1.0/24`.

You can use the `kubectl describe networkpolicy restrict-cidr` command to check this policy:

```bash
kubectl describe networkpolicy restrict-cidr
```

**Output Example**:
```plaintext
Name:         restrict-cidr
Namespace:    default
PodSelector:  app=backend
PolicyTypes:  Ingress
Allowed Ingress:
  - From:
    - IPBlock: 10.0.0.0/16, except 10.0.1.0/24
```

**[Back to TOC](#updated-table-of-contents)**

---

### **6. Working with Labels and Selectors** <a name="working-with-labels-and-selectors"></a>

In network policies, **labels** and **selectors** are used to identify the Pods that the policy applies to. However, students sometimes get confused about multiple occurrences of `podSelector` in a policy file—especially when dealing with both the target Pods and the source Pods (for ingress traffic, for example).

There are two primary levels where `podSelector` is used:
1. **At the top level**: This defines which Pods the network policy is **applied to**.
2. **Within the ingress/egress rules**: This identifies the Pods **from which traffic is allowed or denied**.

Here’s an example policy with multiple `podSelector` entries:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend-and-logging
  namespace: default
spec:
  # This podSelector selects the Pods the policy is applied to (in this case, the backend Pods)
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    # This podSelector defines the Pods from which ingress traffic is allowed (frontend Pods)
    podSelector:
      matchLabels:
        app: frontend
  - from:
    # Another podSelector defines the Pods from which ingress traffic is allowed (logging Pods)
    podSelector:
      matchLabels:
        app: logging
  egress:
  - to:
    # This podSelector defines the Pods to which egress traffic is allowed (database Pods)
    podSelector:
      matchLabels:
        app: database
```

#### **Explanation of the `podSelector` confusion:**
- **Top-level `podSelector`**: 
    - This is used to specify which Pods the policy applies to. In the example above, the policy applies to the **backend Pods** (because of the `matchLabels: app: backend`).
- **Ingress `podSelector`**: 
    - This is used to define **where the traffic is allowed to come from**. In the example, traffic is allowed **from the frontend and logging Pods** (`matchLabels: app: frontend` and `app: logging`).
- **Egress `podSelector`**: 
    - This defines **where the traffic is allowed to go to**. Here, the backend Pods are allowed to send traffic **to the database Pods** (`matchLabels: app: database`).

<a name="example-of-confusion-with-multiple-podselectors"></a>**Common Confusion**:
- Students often mistake the top-level `podSelector` for the one in the ingress or egress rules. **The top-level `podSelector` applies the policy to the Pods (backend), while the lower-level `podSelector` defines the source or destination of the traffic (frontend, logging, database)**.

**Verify the policy using `kubectl describe`:**

```bash
kubectl describe networkpolicy allow-frontend-to-backend-and-logging
```

**Output Example**:

```plaintext
Name:         allow-frontend-to-backend-and-logging
Namespace:    default
PodSelector:  app=backend
PolicyTypes:  Ingress, Egress
Allowed Ingress:
  - From:
    - PodSelector: app=frontend
  - From:
    - PodSelector: app=logging
Allowed Egress:
  - To:
    - PodSelector: app=database
```

This output shows that the policy applies to the backend Pods (`PodSelector: app=backend`), and allows ingress traffic from the frontend and logging Pods, while allowing egress traffic to the database Pods.

<a name="what-is-the-difference-between-the-podselector-at-the-top-level-of-a-network-policy-and-the-one-used-inside-the-ingress-egress-rules"></a>**Interview Question**:  
- **What is the difference between the `podSelector` at the top level of a Network Policy and the one used inside the ingress/egress rules?**
    - The top-level `podSelector` defines which Pods the policy applies to, while the `podSelector` inside ingress/egress rules defines the source or destination Pods for the traffic.

**[Back to TOC](#updated-table-of-contents)**

---

### **7. Using Namespace Selectors** <a name="using-namespace-selectors"></a>

You can also apply policies based on namespaces. This is useful if you want to isolate certain namespaces or limit communication between namespaces.

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: namespace-isolation
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: frontend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          environment: dev
```

**Explanation**:
- This allows ingress traffic to the frontend Pods in the `production` namespace only from Pods in the `dev` namespace.

**[Back to TOC](#updated-table-of-contents)**

---

### **8. Working with IP Blocks** <a name="working-with-ip-blocks"></a>

IP blocks can be used in network policies to allow or deny traffic based on IP addresses.

For example:
```yaml
- ipBlock:
    cidr: 10.0.0.0/16
    except:
    - 10.0.1.0/24
```

In this configuration:
- Traffic is allowed from `10.0.0.0/16` but denied from `10.0.1.0/24`.

**[Back to TOC](#updated-table-of-contents)**

---

### **9. Practical Usage in Cloud and Common Issues** <a name="practical-usage-in-cloud-and-common-issues"></a>

When working with network policies in cloud-managed Kubernetes clusters, the behavior of network policies may differ depending on the provider. This section will explain how to enable and work with network policies in **Google Kubernetes Engine (GKE)**, **Azure Kubernetes Service (AKS)**, and **Amazon Elastic Kubernetes Service (EKS)**.

#### **Google Kubernetes Engine (GKE)**

GKE offers two types of clusters: **Standard** and **Autopilot**. Here’s how network policies differ between them:

1. **Autopilot Cluster**:
   - In Autopilot mode, **network policies are enabled by default**. This means you can immediately apply your network policies through YAML files, and they will function as expected without additional setup.
   
2. **Standard Cluster**:
   - In Standard GKE clusters, network policies are **not enabled by default**. You cannot enable network policies via the GUI (dropdown or click selection). Instead, you must use a command-line interface to enable network policies.

   **Command to enable network policies on a Standard GKE cluster**:
   ```bash
   gcloud container clusters update my-cluster --update-addons=NetworkPolicy=ENABLED --zone my-zone
   ```

   Once enabled, you can apply network policies to your Pods.

#### **Azure Kubernetes Service (AKS)**

In **Azure Kubernetes Service (AKS)**, network policies are not enabled by default either. However, AKS supports both **Azure network policies** and **Kubernetes network policies**. Here's how you can work with them:

1. **Kubernetes Network Policies**:
   - If you want to use Kubernetes-native network policies, you must enable the `networkPolicy` option when creating the cluster.
   
   **Command to enable Kubernetes network policies in AKS**:
   ```bash
   az aks create --resource-group myResourceGroup --name myAKSCluster --network-policy azure
   ```

2. **Azure Network Policies**:
   - Azure-specific network policies are supported, and you can choose between **Azure** or **Kubernetes** policy options. By default, Azure network policies may be more tightly integrated with the Azure networking stack.

#### **Amazon Elastic Kubernetes Service (EKS)**

In **Amazon EKS**, network policies also require additional setup because Kubernetes network policies are not enabled by default. EKS provides flexibility in how network policies are managed, but it requires using a **CNI plugin** such as **Calico** to implement network policies.

1. **Enable Calico for Network Policies**:
   - To enable network policies in EKS, you typically need to install and configure the Calico CNI plugin.
   
   **Command to install Calico in EKS**:
   ```bash
   kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
   ```

   Once Calico is installed, you can apply network policies as you would in a regular Kubernetes setup.

#### **Comparison Summary**

- **GKE Autopilot**: Network policies enabled by default.
- **GKE Standard**: Requires a command-line command to enable network policies.
- **AKS**: Network policies must be enabled during cluster creation, and you can choose between Kubernetes and Azure policies.
- **EKS**: Requires the installation of a CNI plugin like Calico to enable network policies.

#### **Interview Question** <a name="question-cloud-policy"></a>  
- **In cloud-managed Kubernetes clusters, what differences exist between enabling network policies in GKE, AKS, and EKS?**
  
  **Answer**:
  - In **GKE**:
    - **Autopilot clusters** have network policies enabled by default.
    - **Standard clusters** require the command-line activation of network policies (`gcloud container clusters update ...`).
  - In **AKS**:
    - You must specify the `--network-policy` option at the time of cluster creation, allowing you to choose between Azure and Kubernetes network policies.
  - In **EKS**:
    - Network policies are not enabled by default, but you can install the **Calico CNI plugin** to enable them.


**[Back to TOC](#table-of-contents)**


---

### **10. Conclusion** <a name="conclusion"></a>

Network Policies in Kubernetes provide a robust mechanism to control traffic flow between Pods, services, and even external sources. They help enhance security, reduce unnecessary traffic, and optimize network resource usage. By mastering network policies, you can ensure better isolation and security within your Kubernetes cluster, a critical aspect for production environments.

**[Back to TOC](#updated-table-of-contents)**

