
### ETCD Topology: Stacked vs External
In a Stacked ETCD Topology the etcd storage cluster and the Kubernetes control plane share the same physical nodes. It contrasts with an External ETCD Topology, where etcd nodes are separate and dedicated only to running etcd, independent of the control plane nodes.

### Ways to Determine Topology of ETCD
1. Is etcd pod is running on control plane.
2. Is api-serve server for etcd pointing to localhost or control plane for communication.

#### Pod Running on Control Plane

   ```
   kubectl get pods -n kube-system
   ```

![image](https://github.com/user-attachments/assets/da30b509-295a-4b15-94e3-9dc79a2d1e68)

Since etcd pod is running on the controlplane node it is therefore a stacked topology.

#### Kube-Apiserver pointing to local host or control plane for communication.

   ```
   kubectl describle pod kube-apiserver-cluster1-controlplane -n kube-system
   ```

   ![image](https://github.com/user-attachments/assets/9bc81a83-9067-4833-9304-c9b82cf3eab0)

   Pointing to a localhost
