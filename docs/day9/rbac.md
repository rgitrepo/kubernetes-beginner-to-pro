
## Role-Based Access Control (RBAC)

**Objective**: This tutorial aims to provide a comprehensive understanding of Role-Based Access Control (RBAC) in Kubernetes, including its configuration, use cases, and practical examples. By the end of this tutorial, you should be able to configure and manage RBAC for your Kubernetes clusters effectively.

### Table of Contents
1. [Introduction to RBAC](#1-introduction-to-rbac)
2. [Authorization Modes in Kubernetes](#2-authorization-modes-in-kubernetes)
3. [Core Components of RBAC](#3-core-components-of-rbac)
   - [Roles and ClusterRoles](#31-roles-and-clusterroles)
     - [Example Role YAML](#example-role-yaml)
     - [Example ClusterRole YAML](#example-clusterrole-yaml)
   - [RoleBindings and ClusterRoleBindings](#32-rolebindings-and-clusterrolebindings)
     - [Example RoleBinding YAML](#example-rolebinding-yaml)
     - [Example ClusterRoleBinding YAML](#example-clusterrolebinding-yaml)
     - [Example of Binding a Role with ClusterRoleBinding](#example-of-binding-a-role-with-clusterrolebinding)
4. [Creating and Applying Roles and Bindings](#4-creating-and-applying-roles-and-bindings)
5. [Permissions, Authorizations, and Authentications](#5-permissions-authorizations-and-authentications)
   - [RoleBinding: Binding a Role within a Namespace](#51-rolebinding-binding-a-role-within-a-namespace)
   - [ClusterRoleBinding: Binding a ClusterRole across the Cluster](#52-clusterrolebinding-binding-a-clusterrole-across-the-cluster)
   - [RoleBinding: Binding a Role to a Service Account within a Namespace](#53-rolebinding-binding-a-role-to-a-service-account-within-a-namespace)
   - [ClusterRoleBinding: Binding a Namespace-Specific Role across the Cluster](#54-clusterrolebinding-binding-a-namespace-specific-role-across-the-cluster)
   - [Authentication and Authorization in Kubernetes](#55-authentication-and-authorization-in-kubernetes)
6. [Wildcard Usage in RBAC](#6-wildcard-usage-in-rbac)
7. [Validating RBAC Configurations](#7-validating-rbac-configurations)
8. [Best Practices for RBAC](#8-best-practices-for-rbac)
9. [Troubleshooting and Common Errors](#9-troubleshooting-and-common-errors)

---

### 1. [Introduction to RBAC](#1-introduction-to-rbac)

**RBAC** (Role-Based Access Control) in Kubernetes is a method for controlling access to the Kubernetes API based on the roles assigned to users or service accounts. Each role consists of rules that define the permissible actions on resources within the cluster.

[Back to TOC](#table-of-contents)

---

### 2. [Authorization Modes in Kubernetes](#2-authorization-modes-in-kubernetes)

Kubernetes supports several authorization modes, which determine how access is granted within the cluster:

- **AlwaysAllow**: Grants access to every request made to the API server. This mode is generally used in non-production environments or for auditing purposes.
- **AlwaysDeny**: Denies all requests, typically used for clusters that store sensitive information and need to ensure no unauthorized access.
- **RBAC**: This mode allows the use of RBAC for fine-grained control over who can do what within the cluster.
- **ABAC (Attribute-Based Access Control)**: This older method provides a more flexible but less structured approach than RBAC, where access is granted based on policies defined in a file.

[Back to TOC](#table-of-contents)

---

### 3. [Core Components of RBAC](#3-core-components-of-rbac)

RBAC in Kubernetes revolves around four main components: Roles, ClusterRoles, RoleBindings, and ClusterRoleBindings.

<div style="text-align: center;">
     <img src="../../pics/clusterrole-and-role-bindings.png" alt="ClusterRole & RoleBindings" style="width: 500px; height: 300px;">
</div>

#### 3.1. [Roles and ClusterRoles](#31-roles-and-clusterroles)

- **Roles**: Define permissions within a specific **namespace**.
- **ClusterRoles**: Define permissions **cluster-wide**.

##### [Example Role YAML](#example-role-yaml)

Here’s an example of a `Role` that grants read access to all pods within a specific namespace:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: my-namespace
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

**Explanation**:

- `apiVersion: rbac.authorization.k8s.io/v1`: Specifies the API version.
- `kind: Role`: Defines that this resource is a `Role`.
- `metadata.namespace: my-namespace`: Defines the namespace where this role is applicable.
- `metadata.name: pod-reader`: Names the `Role`.
- `rules`: Defines what actions can be performed on which resources. Here, it allows reading (`get`, `watch`, `list`) on all pods in the `my-namespace`.

##### [Example ClusterRole YAML](#example-clusterrole-yaml)

Here’s an example of a `ClusterRole` that grants read access to all pods across the entire cluster:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

**Explanation**:

- `apiVersion: rbac.authorization.k8s.io/v1`: Specifies the API version.
- `kind: ClusterRole`: Defines that this resource is a `ClusterRole`.
- `metadata.name: cluster-pod-reader`: Names the `ClusterRole`.
- `rules`: Defines what actions can be performed on which resources. Here, it allows reading (`get`, `watch`, `list`) on all pods in the cluster.

#### 3.2. [RoleBindings and ClusterRoleBindings](#32-rolebindings-and-clusterrolebindings)

- **RoleBindings**: Grant the permissions defined in a Role to a user or service account within a **namespace**. Note that `RoleBinding` can only be used with a `Role`, and it does not work with a `ClusterRole`.

- **ClusterRoleBindings**: Grant the permissions defined in a `ClusterRole` to a user or service account across the **entire cluster**. Unlike `RoleBinding`, `ClusterRoleBinding` is versatile and can bind either a `ClusterRole` or a `Role` (effectively elevating the `Role`'s permissions to cluster-wide scope).

##### [Example RoleBinding YAML](#example-rolebinding-yaml)

Here’s an example of a `RoleBinding` that binds the `Role` from the previous example to a user within a namespace:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: my-namespace
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

**Explanation**:

- `kind: RoleBinding`: Defines that this resource is a `RoleBinding`.
- `metadata.name: read-pods`: Names the `RoleBinding`.
- `metadata.namespace: my-namespace`: Defines the namespace where this binding applies.
- `subjects`: Specifies who the binding applies to. In this case, it’s a user named `jane`.
- `roleRef`: Refers to the `Role` named `pod-reader` that was defined earlier.

##### [Example ClusterRoleBinding YAML](#example-clusterrolebinding-yaml)

Here’s an example of a `ClusterRoleBinding` that binds the `ClusterRole` from the previous example to a user across the cluster:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-pods-cluster-wide
subjects:
- kind: User
  name: john
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-pod-reader
  apiGroup: rbac.authorization.k8s.io
```

**Explanation**:

- `kind: ClusterRoleBinding`: Defines that this resource is a `ClusterRoleBinding`.
- `metadata.name: read-pods-cluster-wide`: Names the `ClusterRoleBinding`.
- `subjects`: Specifies who the binding applies to. In this case, it’s a user named `john`.
- `roleRef`: Refers to the `ClusterRole` named `cluster-pod-reader` that was defined earlier.

##### [Example of Binding a Role with ClusterRoleBinding](#example-of-binding-a-role-with-clusterrolebinding)

Here's an example where `ClusterRoleBinding` binds a `Role` (not a `ClusterRole`) to a user across the cluster:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-pods-namespace-wide
subjects:
- kind: User
  name: alice
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind:

 Role
  name: pod-reader
  namespace: my-namespace
  apiGroup: rbac.authorization.k8s.io
```

**Explanation**:

- `kind: ClusterRoleBinding`: This is a `ClusterRoleBinding`, which generally works with `ClusterRole`, but in this example, it is binding a `Role`.
- `roleRef.kind: Role`: Refers to a `Role` instead of a `ClusterRole`.
- `roleRef.namespace: my-namespace`: Specifies that the `Role` resides in the `my-namespace` namespace. However, since it's bound using a `ClusterRoleBinding`, the permissions will apply across the entire cluster.

[Back to TOC](#table-of-contents)

---

### 4. [Creating and Applying Roles and Bindings](#4-creating-and-applying-roles-and-bindings)

Roles and bindings can be created and applied using `kubectl` commands or by applying YAML manifests. Here is how you can create a `ClusterRole` and bind it:

```bash
kubectl apply -f clusterrole.yaml
kubectl apply -f clusterrolebinding.yaml
```

To verify that the `ClusterRole` and `ClusterRoleBinding` were created successfully, you can use:

```bash
kubectl get clusterroles
kubectl get clusterrolebindings
```

**Expected Output**:

```bash
NAME                    AGE
cluster-pod-reader      2m
...

NAME                        AGE
read-pods-cluster-wide      2m
...
```

These commands list all `ClusterRoles` and `ClusterRoleBindings` in the cluster, allowing you to confirm that they were applied correctly.

To create and bind a `Role`, you can use the following commands:

```bash
kubectl apply -f role.yaml
kubectl apply -f rolebinding.yaml
```

To verify that the `Role` and `RoleBinding` were created successfully, you can use:

```bash
kubectl get roles -n my-namespace
kubectl get rolebindings -n my-namespace
```

**Expected Output**:

```bash
NAME            AGE
pod-reader      2m
...

NAME            AGE
read-pods       2m
...
```

These commands list all `Roles` and `RoleBindings` within the specified namespace, allowing you to confirm that they were applied correctly.

[Back to TOC](#table-of-contents)

---

### 5. [Permissions, Authorizations, and Authentications](#5-permissions-authorizations-and-authentications)

After applying the various bindings (`RoleBindings` and `ClusterRoleBindings`) described in this tutorial, specific permissions are granted to the users or service accounts specified in those bindings. Here’s what kind of permissions or authorizations become available and to whom:

#### 5.1. [RoleBinding: Binding a Role within a Namespace](#51-rolebinding-binding-a-role-within-a-namespace)

##### Example:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: my-namespace
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

**Permissions Granted:**

- **User:** `jane`
- **Permissions:** `get`, `watch`, `list` actions on all `pods`
- **Scope:** Only within the `my-namespace` namespace

**Explanation:** The `RoleBinding` grants user `jane` the ability to read (`get`, `watch`, `list`) all pods within the `my-namespace` namespace. This means `jane` can view and monitor pods in that namespace but cannot create, delete, or modify them.

#### 5.2. [ClusterRoleBinding: Binding a ClusterRole across the Cluster](#52-clusterrolebinding-binding-a-clusterrole-across-the-cluster)

##### Example:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-pods-cluster-wide
subjects:
- kind: User
  name: john
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-pod-reader
  apiGroup: rbac.authorization.k8s.io
```

**Permissions Granted:**

- **User:** `john`
- **Permissions:** `get`, `watch`, `list` actions on all `pods`
- **Scope:** Across the entire Kubernetes cluster

**Explanation:** The `ClusterRoleBinding` grants user `john` the ability to read (`get`, `watch`, `list`) all pods across the entire cluster. This means `john` can view and monitor pods in any namespace within the cluster, but again, cannot create, delete, or modify them.

#### 5.3. [RoleBinding: Binding a Role to a Service Account within a Namespace](#53-rolebinding-binding-a-role-to-a-service-account-within-a-namespace)

##### Example:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-sa
  namespace: my-namespace
subjects:
- kind: ServiceAccount
  name: my-service-account   # Name is case sensitive
  namespace: my-namespace
roleRef:
  kind: Role                 #this must be Role or ClusterRole
  name: pod-reader           # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```

**Permissions Granted:**

- **Service Account:** `my-service-account`
- **Permissions:** `get`, `watch`, `list` actions on all `pods`
- **Scope:** Only within the `my-namespace` namespace

**Explanation:** The `RoleBinding` grants the service account `my-service-account` the ability to read (`get`, `watch`, `list`) all pods within the `my-namespace` namespace. This service account could be used by applications or controllers running in the `my-namespace` to interact with the Kubernetes API and perform read operations on pods.

#### 5.4. [ClusterRoleBinding: Binding a Namespace-Specific Role across the Cluster](#54-clusterrolebinding-binding-a-namespace-specific-role-across-the-cluster)

##### Example:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-pods-namespace-wide
subjects:
- kind: User
  name: alice
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  namespace: my-namespace
  apiGroup: rbac.authorization.k8s.io
```

**Permissions Granted:**

- **User:** `alice`
- **Permissions:** `get`, `watch`, `list` actions on all `pods`
- **Scope:** Across the entire cluster, but using the `Role` defined in `my-namespace`

**Explanation:** This `ClusterRoleBinding` grants user `alice` the ability to read (`get`, `watch`, `list`) all pods across the entire cluster, but using the permissions defined in the `pod-reader` Role, which was originally scoped to `my-namespace`. Despite the Role being defined in a specific namespace, the `ClusterRoleBinding` elevates the scope of these permissions to the entire cluster.

#### 5.5. [Authentication and Authorization in Kubernetes](#55-authentication-and-authorization-in-kubernetes)

- **Authentication:** Kubernetes assumes that the users (`jane`, `john`, `alice`) or service accounts (`my-service-account`) have been authenticated (i.e., their identity has been verified). This process typically involves using certificates, tokens, or other mechanisms.
- **Authorization:** Once authenticated, Kubernetes checks what the user or service account is authorized to do based on the RBAC policies (Roles, RoleBindings, ClusterRoles, ClusterRoleBindings) defined for them.

[Back to TOC](#table-of-contents)

--- 


### 6. [Wildcard Usage in RBAC](#6-wildcard-usage-in-rbac)

Wildcards can be used in RBAC to grant permissions broadly:

- `*` in the `resources` field grants permissions on all resources.
- `*` in the `verbs` field allows all actions (e.g., get, create, update).

**Example**: Granting a user full access to all resources:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: my-namespace
  name: admin
rules:
- apiGroups: [""]
  resources: ["*"]
  verbs: ["*"]
```

[Back to TOC](#table-of-contents)

---

### 7. [Validating RBAC Configurations](#7-validating-rbac-configurations)

To validate whether a specific action is allowed under a given role, Kubernetes provides the `kubectl auth can-i` command.

**Example**:

```bash
kubectl auth can-i create pods --as=jane -n my-namespace
```

This command checks whether the user `jane` can create pods in the `my-namespace`.

[Back to TOC](#table-of-contents)

---

### 8. [Best Practices for RBAC](#8-best-practices-for-rbac)

- **Least Privilege**: Always assign the least privilege necessary. Use RoleBindings rather than ClusterRoleBindings where possible to limit the scope.
- **Namespace Isolation**: Use namespaces to isolate resources and permissions.
- **RoleBinding Over ClusterRoleBinding**: Prefer RoleBindings to limit permissions to a specific namespace rather than across the entire cluster

.

[Back to TOC](#table-of-contents)

---

### 9. [Troubleshooting and Common Errors](#9-troubleshooting-and-common-errors)

- **401 Unauthorized**: This error indicates that the user is not authenticated.
- **403 Forbidden**: This indicates that the user is authenticated but does not have permission to perform the action.
- **Invalid Role or Binding**: Ensure that roles are created before bindings and that the role names and namespaces match.

**Example** of troubleshooting command:

```bash
kubectl describe rolebinding read-pods -n my-namespace
```

This command will show you the details of the RoleBinding, helping to identify any misconfigurations.

[Back to TOC](#table-of-contents)
