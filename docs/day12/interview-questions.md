
### **Table of Contents**

1. [What is the difference between Continuous Deployment and Continuous Delivery?](#1-what-is-the-difference-between-continuous-deployment-and-continuous-delivery)
2. [What happens if a master node goes down in Kubernetes?](#2-what-happens-if-a-master-node-goes-down-in-kubernetes)
3. [How can you calculate quorum and what is the concept of fault tolerance in etcd?](#3-how-can-you-calculate-quorum-and-what-is-the-concept-of-fault-tolerance-in-etcd)

---

### **1. What is the difference between Continuous Deployment and Continuous Delivery?**

- Both Continuous Deployment and Continuous Delivery are implemented in most companies. The primary difference between them is that in Continuous Deployment, everything happens in an **automated fashion** with **no human intervention**. On the other hand, in Continuous Delivery, there is **manual intervention** involved.
- In Continuous Deployment, the code is automatically pushed to production as soon as it passes all tests, while in Continuous Delivery, the code is prepared to be released to production, but a manual approval is required before deployment.

[Back to TOC](#table-of-contents)

---

### **2. What happens if a master node goes down in Kubernetes?**

- If the master node goes down, **the application running on the worker nodes will not be affected**. However, **new features cannot be rolled out** because critical components like the **API server**, **etcd**, **controller manager**, and **scheduler** running on the master node are down.
- The **API server** handles all requests to the Kubernetes control plane, the **etcd** stores the cluster state, the **controller manager** ensures the desired state of resources, and the **scheduler** places new Pods on the appropriate worker nodes.
- Existing applications will keep running, but **no new deployments** or **scaling** can happen until the master node is restored.

[Back to TOC](#table-of-contents)

---

### **3. How can you calculate quorum and what is the concept of fault tolerance in etcd?**

- **Quorum Formula**:  
  Quorum = `floor(N / 2) + 1`, where `N` is the total number of nodes in the etcd cluster.
  
- **Fault Tolerance**:  
  Fault tolerance refers to how many nodes can fail while the system remains operational.  
  Fault Tolerance = `Total Nodes - Quorum`
  
- **Example**:  
  For a 5-node etcd cluster:  
  - Quorum = `3`
  - Fault tolerance = `2`
  This means the cluster can tolerate 2 node failures and still maintain quorum and functionality.

[Back to TOC](#table-of-contents)

