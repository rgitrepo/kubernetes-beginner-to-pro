
### ETCD Topology: Stacked vs External
In a Stacked ETCD Topology the etcd storage cluster and the Kubernetes control plane share the same physical nodes. It contrasts with an External ETCD Topology, where etcd nodes are separate and dedicated only to running etcd, independent of the control plane nodes.

### Ways to Determine Topology of ETCD
1.  ETCD pod is running on control plane.
3. Api-serve server for etcd pointing to localhost or control plane for communication.

#### 1.1 Pod Running on Control Plane - looking at pods

   ```
   kubectl get pods -n kube-system
   ```

![image](https://github.com/user-attachments/assets/da30b509-295a-4b15-94e3-9dc79a2d1e68)

Since etcd pod is running on the controlplane node it is therefore a stacked topology.

#### 1.1 Pod Running on Control Plane - logging into controlplane node in same context of cluster and looking for static pod files for etcd

Ensure to have the right context by using `kubectl config use-context cluster1` as an example. 

```
student-node ~ kubectl get nodes
NAME                    STATUS   ROLES           AGE    VERSION
cluster1-controlplane   Ready    control-plane   102m   v1.29.0
cluster1-node01         Ready    <none>          101m   v1.29.0
```

```
student-node ~ ➜  ssh cluster1-controlplane
Welcome to Ubuntu 18.04.6 LTS (GNU/Linux 5.4.0-1106-gcp x86_64)
```

Now we check if etcd file is there for static pod:
```
cluster1-controlplane ~ ➜  ls /etc/kubernetes/manifests/
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml
```

There is a file etcd.yaml for static pod that's on control plane node.

#### 2. Kube-Apiserver etcd IP address pointing to local host or to ip of control plane node (server) for communication.

   ```
   kubectl describle pod kube-apiserver-cluster1-controlplane -n kube-system
   ```

   ![image](https://github.com/user-attachments/assets/9bc81a83-9067-4833-9304-c9b82cf3eab0)

   Pointing to a localhost
