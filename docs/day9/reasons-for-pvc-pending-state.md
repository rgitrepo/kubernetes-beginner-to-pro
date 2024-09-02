# Reasons for PVC Pending State in Kubernetes

### Table of Contents

0. [Three States of PVC](#three-states-of-pvc)
1. [Access Mode Mismatch](#1-access-mode-mismatch)
2. [Size Mismatch](#2-size-mismatch)
3. [StorageClass Issues](#3-storageclass-issues)
4. [No Available PVs Matching the PVC](#4-no-available-pvs-matching-the-pvc)
5. [Insufficient Resources](#5-insufficient-resources)
6. [Binding Conflict](#6-binding-conflict)

---
### Three States of a PVC
A Persistent Volume Claim (PVC) in Kubernetes can be in one of three states:

Pending:
The PVC is created, but it has not yet been bound to a PV. This typically occurs when no suitable PV is available that meets the PVC's requirements.
Bound:
The PVC has successfully found a matching PV and is now bound to it. The storage is available for use by the pod.
Lost:
This state occurs when a PVC was previously bound to a PV, but the PV is no longer available or has been deleted. This might happen if the underlying storage is removed or if there’s a configuration error.

---

### 1. Access Mode Mismatch

One of the most common reasons a PVC remains in a pending state is an access mode mismatch between the PVC and the available PVs.

- **Scenario**: A PVC requests `ReadWriteOnce` (RWO) access mode, but the available PVs are configured with `ReadWriteMany` (RWX).
  - **RWO**: The volume can be mounted as read-write by a single node.
  - **RWX**: The volume can be mounted as read-write by multiple nodes simultaneously.
  
- **Example**:
  ```yaml
  # Persistent Volume Claim requesting RWO
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: pvc-rwo
  spec:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 10Gi
    storageClassName: standard
  ```

  ```yaml
  # Persistent Volume with RWX
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: pv-rwx
  spec:
    capacity:
      storage: 20Gi
    accessModes:
      - ReadWriteMany
    persistentVolumeReclaimPolicy: Retain
    storageClassName: standard
    hostPath:
      path: "/mnt/data"
  ```

  - **Output**: When you describe the PVC, you might see something like this:
    ```
    kubectl describe pvc pvc-rwo
    ```

    ```
    Name:          pvc-rwo
    Namespace:     default
    Status:        Pending
    Volume:        
    Labels:        <none>
    Annotations:   <none>
    Finalizers:    [kubernetes.io/pvc-protection]
    StorageClass:  standard
    Access Modes:  RWO
    VolumeMode:    Filesystem
    Capacity:      10Gi
    Conditions:
      Type    Status  LastProbeTime                     LastTransitionTime                Reason        Message
      ----    ------  -----------------                 ------------------                ------        -------
      Bound   False   <unknown>                         <unknown>                         VolumeBinding  waiting for a volume to be created or to be bound to a volume
    ```

    The PVC remains in the **Pending** state because there is no PV with an access mode of `ReadWriteOnce` (RWO) available, only `ReadWriteMany` (RWX). Kubernetes cannot bind the PVC to a PV that doesn’t meet its access mode requirements.

[Back to Table of Contents](#table-of-contents)

---

### 2. Size Mismatch

Another common issue arises when the PVC requests more storage than any available PV can provide.

- **Scenario**: A PVC requests 20Gi of storage, but all available PVs offer only 10Gi.
  
- **Example**:
  ```yaml
  # Persistent Volume Claim requesting 20Gi
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: pvc-large
  spec:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 20Gi
    storageClassName: standard
  ```

  ```yaml
  # Persistent Volume with only 10Gi
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: pv-small
  spec:
    capacity:
      storage: 10Gi
    accessModes:
      - ReadWriteOnce
    persistentVolumeReclaimPolicy: Retain
    storageClassName: standard
    hostPath:
      path: "/mnt/data"
  ```

  - **Output**: When describing the PVC, you might see the following:
    ```
    kubectl describe pvc pvc-large
    ```

    ```
    Name:          pvc-large
    Namespace:     default
    Status:        Pending
    Volume:        
    Labels:        <none>
    Annotations:   <none>
    Finalizers:    [kubernetes.io/pvc-protection]
    StorageClass:  standard
    Access Modes:  RWO
    VolumeMode:    Filesystem
    Capacity:      20Gi
    Conditions:
      Type    Status  LastProbeTime                     LastTransitionTime                Reason        Message
      ----    ------  -----------------                 ------------------                ------        -------
      Bound   False   <unknown>                         <unknown>                         VolumeBinding  waiting for a volume to be created or to be bound to a volume
    ```

    The PVC remains **Pending** because there is no PV with sufficient capacity (20Gi) available. Kubernetes cannot bind a PVC to a PV that doesn't meet its storage requirements.

[Back to Table of Contents](#table-of-contents)

---

### 3. StorageClass Issues

StorageClass-related problems can also prevent a PVC from being bound to a PV, leaving it in a Pending state.

- **No Available StorageClass**: If the PVC specifies a `StorageClass` that does not exist or is not properly configured in the cluster, Kubernetes will not be able to provision a PV, resulting in the PVC staying in a Pending state.
- **Default StorageClass Misconfiguration**: If a PVC does not specify a `StorageClass` and there is no default `StorageClass` configured, Kubernetes will not know how to dynamically provision a PV, leaving the PVC in Pending status.

  **Example**:
  ```yaml
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: pvc-missing-class
  spec:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 10Gi
    storageClassName: non-existent-class
  ```

  - **Output**:
    ```
    kubectl describe pvc pvc-missing-class
    ```

    ```
    Type     Status  LastProbeTime                     LastTransitionTime                Reason             Message
    ----     ------  -----------------                 ------------------                ------             -------
    Bound    False   <unknown>                         <unknown>                         StorageClassNotFound  waiting for a storage class to be created or to be bound to a volume
    ```

[Back to Table of Contents](#table-of-contents)

---

### 4. No Available PVs Matching the PVC

Even if access modes and size are correct, the PVC might still remain pending if there are no PVs available that match other criteria specified in the PVC.

- **VolumeMode Mismatch**: A PV might be configured with `VolumeMode: Filesystem`, but the PVC requests `VolumeMode: Block`.
- **Labels or Selectors**: If a PVC specifies labels or selectors that no PV matches, the PVC will stay pending.

  **Example**:
  ```yaml
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: pvc-with-selector
  spec:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 10Gi
    selector:
      matchLabels:
        type: fast-storage
  ```

  - **Output**:
    ```
    kubectl describe pvc pvc-with-selector
    ```

    ```
    Type     Status  LastProbeTime                     LastTransitionTime                Reason                 Message
    ----     ------  -----------------                 ------------------                ------                 -------
    Bound    False   <unknown>                         <unknown>                         VolumeBinding          waiting for a volume to be created or to be bound to a volume
    ```

[Back to Table of Contents](#table-of-contents)

---

### 5. Insufficient Resources

In cases where dynamic provisioning is used, the underlying cloud provider might not have sufficient resources to provision the requested storage. 

- **Scenario**: If an AWS EBS volume is requested but the AWS account has reached its limit for EBS volumes or there are no available disks in the specified zone, the PVC will remain pending.

  **Output**:
    ```
    kubectl describe pvc dynamic-pvc
    ```

    ```
    Type     Status  LastProbeTime                     LastTransitionTime                Reason                      Message
    ----     ------  -----------------                 ------------------                ------                      -------
    Bound    False   <unknown>                         <unknown>                         ProvisioningFailed          Failed to provision volume with StorageClass "gp2": no available disk in zone
    ```

[Back to Table of Contents](#table-of-contents)

---

### 6. Binding Conflict

If two PVCs attempt to bind to the same PV that only supports `ReadWriteOnce`, only one will succeed, and the other will remain in the Pending state.

  **Output**:
    ```
    kubectl describe pvc conflicting-pvc
    ```

    ```
    Type     Status  LastProbeTime                     LastTransitionTime                Reason                  Message
    ----     ------  -----------------                 ------------------                ------                  -------
    Bound    False   <unknown>                         <unknown>                         VolumeBinding            waiting for a volume to be created or to be bound to a volume
    ```

[Back to Table of Contents](#table-of-contents)

---

### Conclusion

Understanding the various reasons why a PVC might remain in a Pending state is crucial for effective storage management in Kubernetes. By considering access mode mismatches, size mismatches, StorageClass issues, availability of matching PVs, resource constraints, and potential binding conflicts, you can troubleshoot and resolve PVC issues more efficiently.
