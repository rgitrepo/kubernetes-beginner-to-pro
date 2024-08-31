Here is the updated version with working links:

### Interview Questions - Table of Contents

1. [What are the differences between Authentication (AuthN) and Authorization (AuthZ)?](#question-1-what-are-the-differences-between-authentication-authn-and-authorization-authz)
2. [Which Kubernetes component is responsible for both authentication and authorization?](#question-2-which-kubernetes-component-is-responsible-for-both-authentication-and-authorization)
3. [What is the purpose of the `kubeconfig` file?](#question-3-what-is-the-purpose-of-the-kubeconfig-file)
4. [Where is the `kubeconfig` file typically located?](#question-4-where-is-the-kubeconfig-file-typically-located)
5. [What are the key components of a `kubeconfig` file?](#question-5-what-are-the-key-components-of-a-kubeconfig-file)
6. [How can you switch between different contexts in Kubernetes?](#question-6-how-can-you-switch-between-different-contexts-in-kubernetes)
7. [What happens if you create a service account or user account without defining any permissions?](#question-7-what-happens-if-you-create-a-service-account-or-user-account-without-defining-any-permissions)
8. [How can authorization be performed in Kubernetes?](#question-8-how-can-authorization-be-performed-in-kubernetes)
9. [What should you check first when troubleshooting authorization issues in Kubernetes?](#question-9-what-should-you-check-first-when-troubleshooting-authorization-issues-in-kubernetes)
10. [What are the potential issues of running multiple workloads in a single cluster?](#question-10-what-are-the-potential-issues-of-running-multiple-workloads-in-a-single-cluster)
11. [Why is it important to define namespaces or contexts in large organizations?](#question-11-why-is-it-important-to-define-namespaces-or-contexts-in-large-organizations)
12. [If we have namespaces to deal with the Noisy Neighbor problem, why do we need contexts?](#question-12-if-we-have-namespaces-to-deal-with-the-noisy-neighbor-problem-why-do-we-need-contexts)

---

### 1. What are the differences between Authentication (AuthN) and Authorization (AuthZ)?
- **Answer**: 
  - **Authentication (AuthN)**: The process of verifying who you are. For example, when you enter a bank, your identity is verified before you are allowed to access your locker. In Kubernetes, authentication ensures that the entity interacting with the system is who they claim to be.
  - **Authorization (AuthZ)**: The process of determining what you are allowed to do. After authentication, Kubernetes checks whether the authenticated entity has the permissions to perform the requested operation. For example, at an airport, after your identity is verified, you are only allowed to access the counter of the airline you are flying with, not any other airline's counter.
- [Back to TOC](#interview-questions---table-of-contents)

### 2. Which Kubernetes component is responsible for both authentication and authorization?
- **Answer**: 
  - The **Kubernetes API Server** is responsible for handling both authentication and authorization in Kubernetes.
- [Back to TOC](#interview-questions---table-of-contents)

### 3. What is the purpose of the `kubeconfig` file?
- **Answer**: 
  - The `kubeconfig` file is used by Kubernetes to manage cluster configurations, user credentials, and context settings. It allows you to specify which cluster and user credentials to use, and it provides a way to switch between different clusters and contexts.
- [Back to TOC](#interview-questions---table-of-contents)

### 4. Where is the `kubeconfig` file typically located?
- **Answer**: 
  - The `kubeconfig` file is usually located in the `.kube` directory in the user's home directory.
- [Back to TOC](#interview-questions---table-of-contents)

### 5. What are the key components of a `kubeconfig` file?
- **Answer**: 
  - The key components of a `kubeconfig` file are:
    - **Clusters**: Define the endpoint of a Kubernetes cluster and the CA data.
    - **Users**: Contain the authentication credentials for users.
    - **Contexts**: Link a user to a specific cluster along with a default namespace.
    - **Current Context**: Defines the active context that `kubectl` will use by default.
- [Back to TOC](#interview-questions---table-of-contents)

### 6. How can you switch between different contexts in Kubernetes?
- **Answer**: 
  - You can switch between contexts using the command:
    ```bash
    kubectl config use-context <context-name>
    ```
- [Back to TOC](#interview-questions---table-of-contents)

### 7. What happens if you create a service account or user account without defining any permissions?
- **Answer**: 
  - By default, Kubernetes assigns zero permissions to any newly created service account or user account. If you don't explicitly define permissions, the account won't be able to perform any operations.
- [Back to TOC](#interview-questions---table-of-contents)

### 8. How can authorization be performed in Kubernetes?
- **Answer**: 
  - Authorization in Kubernetes can be performed in several ways:
    - **Webhook Authorization**: A dynamic method where the API Server sends authorization requests to an external webhook server.
    - **Static Token File**: Uses a pre-defined file containing tokens and usernames.
    - **X.509 Certificates**: A common method where users authenticate using X.509 client certificates.
    - **OpenID Connect (OIDC)**: An identity layer on top of OAuth 2.0, used to authenticate users with external identity providers.
    - **Custom Authorization**: Allows custom modules for specific use cases.
- [Back to TOC](#interview-questions---table-of-contents)

### 9. What should you check first when troubleshooting authorization issues in Kubernetes?
- **Answer**: 
  - The first thing to check is whether the request was authenticated correctly. After that, verify if the entity is authorized to perform the requested action. If authentication and authorization are correct, proceed to check for any validation or mutation webhooks, policies, or runtime security settings.
- [Back to TOC](#interview-questions---table-of-contents)

### 10. What are the potential issues of running multiple workloads in a single cluster?
- **Answer**: 
  - One potential issue is the **Noisy Neighbor** problem, where one workload consumes excessive resources, leaving insufficient resources for other workloads. This can lead to performance degradation or even application downtime.
- [Back to TOC](#interview-questions---table-of-contents)

### 11. Why is it important to define namespaces or contexts in large organizations?
- **Answer**: 
  - Defining namespaces or contexts in large organizations helps manage resources effectively and prevents issues like the **Noisy Neighbor** problem. It also ensures that different teams or workloads don't interfere with each other, enabling smoother operations and easier management of permissions and access controls.
- [Back to TOC](#interview-questions---table-of-contents)

### 12. If we have namespaces to deal with the Noisy Neighbor problem, why do we need contexts?
- **Answer**: 
  - **Namespaces** help isolate resources within a cluster, ensuring that workloads don't interfere with each other by providing logical boundaries. However, **contexts** go a step further by linking specific users, clusters, and namespaces together, allowing for easier management across different environments and teams. Contexts are especially useful in large organizations where multiple clusters are used for different purposes (e.g., development, staging, production). They allow users to switch between different environments (contexts) seamlessly without manually configuring their `kubeconfig` file each time. This ensures that a team working on a particular cluster or namespace can focus on their work without affecting other teams, even if they are working within the same Kubernetes ecosystem.
- [Back to TOC](#interview-questions---table-of-contents)
