To check if the Kubernetes scheduler is not installed or disabled, you can look for signs such as pods being stuck in the **Pending** state. The scheduler is responsible for assigning pods to nodes, so without it, pods will not be scheduled to any nodes.

### Checking if the Scheduler is Installed or Running

You can verify the status of the scheduler by inspecting the `kube-scheduler` component in the `kube-system` namespace. Here's how to check if it's installed and running:

```bash
kubectl get pods -n kube-system | grep scheduler
```

If the scheduler is functioning correctly, you'll see output similar to this:

```
kube-scheduler-master-node   1/1     Running   0          4h
```

If the scheduler is not running, you may see no output or pods in a failed state.

### Example Output (Scheduler Missing or Not Running)
```
No resources found.
```

You can also check the logs of the scheduler pod (if available) to see if there are any errors or issues:

```bash
kubectl logs kube-scheduler-master-node -n kube-system
```

### Checking the Pod's Status

If pods are in the **Pending** state because the scheduler is disabled or not functioning, you can check their status with:

```bash
kubectl describe pod <pod-name> -n <namespace>
```

Look for events that mention scheduling issues, such as:

```
Events:
  Type     Reason            Age               From               Message
  ----     ------            ----              ----               -------
  Warning  FailedScheduling   2m (x5 over 2m)   default-scheduler  0/5 nodes are available: 5 node(s) didn't match node selector.
```

If the scheduler is disabled or not installed, you may see no scheduling attempts, or scheduling errors may appear.
