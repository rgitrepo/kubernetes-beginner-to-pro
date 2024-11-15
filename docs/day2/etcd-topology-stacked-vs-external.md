
### ETCD Topology: Stacked vs External
In a Stacked ETCD Topology the etcd storage cluster and the Kubernetes control plane share the same physical nodes. It contrasts with an External ETCD Topology, where etcd nodes are separate and dedicated only to running etcd, independent of the control plane nodes.

![image](https://github.com/user-attachments/assets/da30b509-295a-4b15-94e3-9dc79a2d1e68)

Since etcd pod is running on the controlplane node it is therefore a stacked topology.
