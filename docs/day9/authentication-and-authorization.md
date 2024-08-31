### Tutorial on Kubernetes Authentication and Authorization (AuthN & AuthZ)

#### Table of Contents

1. [Introduction](#introduction)
2. [Understanding Authentication (AuthN) and Authorization (AuthZ)](#understanding-authentication-authn-and-authorization-authz)
   - [What is Authentication (AuthN)?](#what-is-authentication-authn)
   - [What is Authorization (AuthZ)?](#what-is-authorization-authz)
3. [Kubernetes Components Handling AuthN & AuthZ](#kubernetes-components-handling-authn--authz)
4. [How Authentication Works in Kubernetes](#how-authentication-works-in-kubernetes)
   - [Client-Side and Server-Side Authentication](#client-side-and-server-side-authentication)
   - [Authentication Flow](#authentication-flow)
5. [How Authorization Works in Kubernetes](#how-authorization-works-in-kubernetes)
   - [Ways to Perform Authorization](#ways-to-perform-authorization)
   - [Role-Based Access Control (RBAC)](#role-based-access-control-rbac)
   - [Authorization Flow](#authorization-flow)
6. [Practical Examples](#practical-examples)
   - [YAML Example for RBAC](#yaml-example-for-rbac)
   - [Applying and Testing RBAC](#applying-and-testing-rbac)
7. [Conclusion](#conclusion)

---

### Introduction

In Kubernetes, Authentication (AuthN) and Authorization (AuthZ) are crucial mechanisms that control access to your cluster resources. Understanding these concepts is essential for managing who can access your cluster and what actions they can perform. This tutorial will explain the concepts of AuthN and AuthZ, how they are implemented in Kubernetes, and provide practical examples to help you understand how to secure your Kubernetes environment.

[Back to TOC](#table-of-contents)

---

### Understanding Authentication (AuthN) and Authorization (AuthZ)

#### What is Authentication (AuthN)?

**Authentication** (often abbreviated as AuthN) is the process of verifying the identity of a user or service. In Kubernetes, authentication ensures that the entity trying to access the cluster is who they claim to be.

**Example**: When you log in to your bank's website, the bank verifies your identity by checking your username and password. Similarly, in Kubernetes, authentication verifies that you are the person or service you claim to be.

#### What is Authorization (AuthZ)?

**Authorization** (often abbreviated as AuthZ) is the process of determining whether a user or service has the right to perform a specific action within the system. After authentication, Kubernetes checks whether the authenticated entity is authorized to perform the requested operation.

**Example**: After verifying your identity, the bank checks whether you have the right to access your account or perform certain transactions. In Kubernetes, authorization checks whether you have the necessary permissions to perform actions like creating a Pod or accessing a resource.

[Back to TOC](#table-of-contents)

---

### Kubernetes Components Handling AuthN & AuthZ

In Kubernetes, the **Kubernetes API Server** is the central component responsible for both authentication and authorization. When you issue a command using `kubectl`, the request first goes through the Kubernetes API Server, which handles both AuthN and AuthZ before allowing any operations to proceed.

**Key Component**:
- **Kubernetes API Server**: Manages all API requests and ensures that each request is authenticated and authorized before processing.

[Back to TOC](#table-of-contents)

---

### How Authentication Works in Kubernetes

#### Client-Side and Server-Side Authentication

Authentication in Kubernetes can occur both on the client-side and server-side.

- **Client-Side Authentication**: This occurs when the client (e.g., `kubectl`) presents credentials (like certificates, tokens, or passwords) to the API Server. The API Server checks these credentials to verify the client's identity.
  
- **Server-Side Authentication**: This is where the API Server itself validates the credentials against configured authentication mechanisms (e.g., static token files, client certificates, or external identity providers).

#### Authentication Flow

1. **User Initiates Request**: A user or service initiates a request using a tool like `kubectl`.
2. **API Server Receives Request**: The Kubernetes API Server receives the request and checks the credentials provided by the client.
3. **Authentication Check**: The API Server verifies the client's identity using one of the supported authentication methods (e.g., client certificates, bearer tokens, OpenID Connect).
4. **Authentication Success/Failure**: If the credentials are valid, the API Server proceeds to the authorization step. If not, the request is denied.

[Back to TOC](#table-of-contents)

---

### How Authorization Works in Kubernetes

#### Ways to Perform Authorization

Kubernetes provides several methods for performing authorization, allowing flexibility in how access control is implemented:

1. **Webhook Authorization**: A dynamic authorization method where the API Server sends the authorization request to an external webhook server. The webhook server processes the request and returns whether the action is allowed or denied.

2. **Static Token File**: Uses a pre-defined file containing tokens and associated usernames. This method is simple but less flexible, as it requires manual updates to the token file whenever a change is needed.

3. **X.509 Certificates**: A common and secure method where users and services authenticate using X.509 client certificates. The certificates are validated by the API Server using a configured certificate authority (CA).

4. **OpenID Connect (OIDC)**: An identity layer on top of the OAuth 2.0 protocol, OIDC allows users to authenticate using external identity providers like Google, Microsoft, or other OAuth 2.0 compatible providers. This is common in large organizations that have centralized identity management.

5. **Custom Authorization**: For more specific use cases, Kubernetes allows for custom authorization modules. These are typically rare and used in environments with unique security requirements.

#### Role-Based Access Control (RBAC)

Kubernetes uses **Role-Based Access Control (RBAC)** to manage authorization. RBAC allows you to define roles that specify what actions are allowed, and then assign those roles to users or groups.

- **Role**: A collection of permissions. For example, a role might allow creating, updating, or deleting Pods.
- **RoleBinding**: Links a role to a user or group, granting them the permissions specified in the role.

#### Authorization Flow

1. **API Server Proceeds to Authorization**: After successful authentication, the API Server checks what the authenticated entity is allowed to do.
2. **Check Against RBAC Policies**: The API Server evaluates the request against RBAC policies to determine if the action is permitted.
3. **Authorization Success/Failure**: If the entity has the necessary permissions, the request is processed. If not, the request is denied.

[Back to TOC](#table-of-contents)

---

### Practical Examples

#### YAML Example for RBAC

Here is a YAML example defining an RBAC role and binding it to a user:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

- **Role**: `pod-reader` allows reading Pods in the `default` namespace.
- **RoleBinding**: `read-pods` binds the `pod-reader` role to the user `jane`.

#### Applying and Testing RBAC

1. **Apply the RBAC Configuration**:
   ```bash
   kubectl apply -f rbac-role.yaml
   ```

2. **Test Access**: As the user `jane`, try to list Pods in the `default` namespace:
   ```bash
   kubectl auth can-i list pods --namespace=default --as=jane
   ```

   If the RBAC role is correctly configured, `jane` should be able to list Pods.

[Back to TOC](#table-of-contents)

---

### Conclusion

Authentication and Authorization (AuthN & AuthZ) are vital components of Kubernetes security, ensuring that only authenticated users can access your cluster and that they can only perform actions they are authorized for. Understanding how these processes work and how to implement them using RBAC and other methods is essential for managing a secure Kubernetes environment.

This tutorial provided an overview of AuthN and AuthZ, different authorization methods, how they are handled in Kubernetes, and practical examples of implementing RBAC.

[Back to TOC](#table-of-contents)
