
## **Network Policies in Kubernetes**

---

### **Table of Contents**
1. [Introduction to Network Policies](#introduction)
2. [Ingress and Egress Traffic Explained](#ingress-egress)
3. [Understanding Network Policies in Kubernetes](#understanding-policies)
4. [How Network Policies Are Applied](#policy-application)
5. [Examples of Network Policies](#policy-examples)
    - [Example 1: Denying All Traffic by Default](#deny-all)
        - [Interview Question: What happens if no network policy is applied to a Pod?](#question-no-policy)
    - [Example 2: Allowing Traffic Only Between Specific Pods](#allow-specific-pods)
        - [Interview Question: How can you restrict Pod-to-Pod communication using network policies?](#question-restrict-pod)
    - [Example 3: Restricting Traffic by CIDR Block](#restrict-cidr)
6. [Working with Labels and Selectors](#labels-selectors)
    - [Interview Question: What is the importance of labels in applying network policies?](#question-labels)
7. [Using Namespace Selectors](#namespace-selectors)
8. [Working with IP Blocks](#ip-blocks)
9. [Practical Usage in Cloud and Common Issues](#cloud-usage)
10. [Conclusion](#conclusion)

---

### **5. Examples of Network Policies** <a name="policy-examples"></a>

#### **Example 1: Denying All Traffic by Default** <a name="deny-all"></a>

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

This command provides detailed information on the applied policy, including which types of traffic are affected and the specific rules in place. It’s very useful for debugging and ensuring that the policy behaves as expected.

<a name="question-no-policy"></a>**Interview Question**:  
- **What happens if no network policy is applied to a Pod?**
    - By default, all traffic (both ingress and egress) is allowed.

**[Back to TOC](#table-of-contents)**

---

#### **Example 2: Allowing Traffic Only Between Specific Pods** <a name="allow-specific-pods"></a>

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

<a name="question-restrict-pod"></a>**Interview Question**:  
- **How can you restrict Pod-to-Pod communication using network policies?**
    - By applying specific network policies, such as the one above, where only selected Pods (like frontend) are allowed to communicate with backend Pods.

**[Back to TOC](#table-of-contents)**

---

#### **Example 3: Restricting Traffic by CIDR Block** <a name="restrict-cidr"></a>

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

**[Back to TOC](#table-of-contents)**

