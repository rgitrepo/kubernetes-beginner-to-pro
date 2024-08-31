
## Kubernetes Role-Based Access Control (RBAC) Tutorial

**Objective**: This tutorial aims to provide a comprehensive understanding of Role-Based Access Control (RBAC) in Kubernetes, including its configuration, use cases, and practical examples. By the end of this tutorial, you should be able to configure and manage RBAC for your Kubernetes clusters effectively.

### Table of Contents
1. [Introduction to RBAC](#1-introduction-to-rbac)
2. [Authorization Modes in Kubernetes](#2-authorization-modes-in-kubernetes)
3. [Core Components of RBAC](#3-core-components-of-rbac)
   - [Roles and ClusterRoles](#31-roles-and-clusterroles)
   - [RoleBindings and ClusterRoleBindings](#32-rolebindings-and-clusterrolebindings)
4. [Creating and Applying Roles and Bindings](#4-creating-and-applying-roles-and-bindings)
5. [Wildcard Usage in RBAC](#5-wildcard-usage-in-rbac)
6. [Validating RBAC Configurations](#6-validating-rbac-configurations)
7. [Best Practices for RBAC](#7-best-practices-for-rbac)
8. [Troubleshooting and Common Errors](#8-troubleshooting-and-common-errors)

---

### 1. [Introduction to RBAC](#introduction-to-rbac)

**RBAC** (Role-Based Access Control) in Kubernetes is a method for controlling access to the Kubernetes API based on the roles assigned to users or service accounts. Each role consists of rules that define the permissible actions on resources within the cluster.

[Back to TOC](#table-of-contents)

---

### 2. [Authorization Modes in Kubernetes](#authorization-modes-in-kubernetes)

Kubernetes supports several authorization modes, which determine how access is granted within the cluster:

- **AlwaysAllow**: Grants access to every request made to the API server. This mode is generally used in non-production environments or for auditing purposes.
- **AlwaysDeny**: Denies all requests, typically used for clusters that store sensitive information and need to ensure no unauthorized access.
- **RBAC**: This mode allows the use of RBAC for fine-grained control over who can do what within the cluster.
- **ABAC (Attribute-Based Access Control)**: This older method provides a more flexible but less structured approach than RBAC, where access is granted based on policies defined in a file.

#### Example of Configuring Authorization Mode

To configure the authorization mode in Kubernetes, you typically need to edit the API server configuration file located at `/etc/kubernetes/manifests/kube-apiserver.yaml`. This directory contains the key manifest files that define the Kubernetes control plane components, including `kube-apiserver`, `kube-controller-manager`, and `kube-scheduler`.

**Important Notes**:
- **Location**: The configuration files for the Kubernetes control plane are located in `/etc/kubernetes/manifests`.
- **Impact of Changes**: Making changes to any files in this directory will automatically trigger a reboot of the respective control plane component. This is because these files are monitored by the `kubelet`, and any updates cause the component to restart, applying the new configuration.
- **systemd Involvement**: These control plane components are managed as static pods by the `kubelet`. However, the underlying processes and services are also managed by `systemd`, which is responsible for starting the `kubelet` and ensuring it runs continuously. Any changes to the control plane files trigger actions managed by both the `kubelet` and `systemd`.

The relevant section of the `kube-apiserver.yaml` file might look like this:

```yaml
- --authorization-mode=Node,RBAC
```

In this example, `Node` and `RBAC` modes are enabled. The API server checks authorization using these modes in left-to-right order.

**Explanation of `authorization-mode=Node,RBAC`:**

- **Node**: This mode is used to authorize kubelets. It ensures that each node in the cluster has access only to the API resources necessary to manage its own workloads and nothing more.
- **RBAC**: Role-Based Access Control mode allows defining granular access controls for users and service accounts.

**Significance of Left-to-Right Evaluation**:

- The modes are evaluated in the order they are listed, from left to right. This means that when a request comes to the API server, it first checks the `Node` authorization mode. If the request is authorized by this mode, the API server does not evaluate the subsequent modes.
- If `Node` authorization does not allow the request, the server then checks `RBAC` to see if the request can be authorized. Other modes such as AlwaysDeny, or ABAC can be added to the authorization-mode separated by commas.

**Tweaking to Read Right-to-Left**:

- By default, Kubernetes processes the authorization modes from left to right, but with some custom configurations or by modifying the order in the `kube-apiserver.yaml`, it could potentially prioritize modes differently. However, this is not typical and should be approached with caution as it may lead to unexpected behavior.

[Back to TOC](#table-of-contents)

---

### 3. [Core Components of RBAC](#core-components-of-rbac)

RBAC in Kubernetes revolves around four main components: Roles, ClusterRoles, RoleBindings, and ClusterRoleBindings.

<div style="text-align: center;">
     <img src="../../pics/clusterrole-and-role-bindings.png" alt="ClusterRole & RoleBindings" style="width: 500px; height: 300px;">
</div>

#### 3.1. [Roles and ClusterRoles](#roles-and-clusterroles)

- **Roles**: Define permissions within a specific **namespace**.
- **ClusterRoles**: Define permissions **cluster-wide**.

##### Example Role YAML

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

##### Example ClusterRole YAML

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

#### 3.2. [RoleBindings and ClusterRoleBindings](#rolebindings-and-clusterrolebindings)

- **RoleBindings**: Grant the permissions defined in a Role to a user or service account within a **namespace**. Note that `RoleBinding` can only be used with a `Role`, and it does not work with a `ClusterRole`.

- **ClusterRoleBindings**: Grant the permissions defined in a `ClusterRole` to a user or service account across the **entire cluster**. Unlike `RoleBinding`, `ClusterRoleBinding` is versatile and can bind either a `ClusterRole` or a `Role` (effectively elevating the `Role`'s permissions to cluster-wide scope).

##### Example RoleBinding YAML

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

##### Example ClusterRoleBinding YAML

Here’s an example of a `ClusterRoleBinding` that binds the `ClusterRole` from the previous example to a user across the

 cluster:

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

##### Example of Binding a Role with ClusterRoleBinding

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
  kind: Role
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

### 4. [Creating and Applying Roles and Bindings](#creating-and-applying-roles-and-bindings)

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

### 5. [Wildcard Usage in RBAC](#wildcard-usage-in-rbac)

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

### 6. [Validating RBAC Configurations](#validating-rbac-configurations)

To validate whether a specific action is allowed under a given role, Kubernetes provides the `kubectl auth can-i` command.

**Example**:

```bash
kubectl auth can-i create pods --as=jane -n my-namespace
```

This command checks whether the user `jane` can create pods in the `my-namespace`.

[Back to TOC](#table-of-contents)

---

### 7. [Best Practices for RBAC](#best-practices-for-rbac)

- **Least Privilege**: Always assign the least privilege necessary. Use RoleBindings rather than ClusterRoleBindings where possible to limit the scope.
- **Namespace Isolation**: Use namespaces to isolate resources and permissions.
- **RoleBinding Over ClusterRoleBinding**: Prefer RoleBindings to limit permissions to a specific namespace rather than across the entire cluster.

[Back to TOC](#table-of-contents)

---

### 8. [Troubleshooting and Common Errors](#troubleshooting-and-common-errors)

- **401 Unauthorized**: This error indicates that the user is not authenticated.
- **403 Forbidden**: This indicates that the user is authenticated but does not have permission to perform the action.
- **Invalid Role or Binding**: Ensure that roles are created before bindings and that the role names and namespaces match.

**Example** of troubleshooting command:

```bash
kubectl describe rolebinding read-pods -n my-namespace
```

This command will show you the details of the RoleBinding, helping to identify any misconfigurations.

[Back to TOC](#table-of-contents)

