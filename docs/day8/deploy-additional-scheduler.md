### Deploying an Additional Kubernetes Scheduler

---

#### Table of Contents
1. [Introduction to Kubernetes Schedulers](#introduction-to-kubernetes-schedulers)
2. [Deploying an Additional Scheduler Using systemd](#deploying-an-additional-scheduler-using-systemd)
   - [Creating a Custom Scheduler Service](#creating-a-custom-scheduler-service)
   - [Setting Up the Scheduler Configuration](#setting-up-the-scheduler-configuration)
   - [Configuring the Custom Scheduler](#configuring-the-custom-scheduler)
3. [Deploying the Scheduler as a Pod](#deploying-the-scheduler-as-a-pod)
   - [Custom Scheduler Pod Configuration](#custom-scheduler-pod-configuration)
4. [Managing Multiple Schedulers in Kubernetes](#managing-multiple-schedulers-in-kubernetes)
5. [Verifying the Custom Scheduler](#verifying-the-custom-scheduler)
   - [Checking Pods](#checking-pods)
   - [Viewing Logs and Events](#viewing-logs-and-events)
6. [Conclusion](#conclusion)

---

### Introduction to Kubernetes Schedulers

By default, Kubernetes uses the **kube-scheduler** to assign pods to nodes. In specific use cases, you may need to deploy additional schedulers for custom pod placement policies. While deploying multiple schedulers is not the most common method these days, it is useful for specialized workloads or experimentation.

---

### Deploying an Additional Scheduler Using systemd

You can deploy an additional scheduler using systemd on a node. Here’s how to set up a new scheduler service.

#### Creating a Custom Scheduler Service

Create a new systemd service file for your custom scheduler (e.g., `my-scheduler.service` or `my-scheduler-2.service`).

1. **Navigate to the systemd service directory**:
   ```bash
   cd /etc/systemd/system/
   ```

2. **Create the service file**:
   ```bash
   wget https://example.com/my-scheduler.service
   ```

3. **Example content for `my-scheduler.service`**:
   ```ini
   [Unit]
   Description=My Custom Kubernetes Scheduler
   Wants=network-online.target
   After=network-online.target

   [Service]
   ExecStart=/usr/local/bin/kube-scheduler \
   --config=/etc/kubernetes/config/my-scheduler-config.yaml

   [Install]
   WantedBy=multi-user.target
   ```

#### Setting Up the Scheduler Configuration

You need a custom `KubeSchedulerConfiguration` YAML file to define how your new scheduler will behave. For example, `my-scheduler-config.yaml`.

**Example Configuration**:
```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: /etc/kubernetes/kubeconfig/my-scheduler-kubeconfig.yaml
leaderElection:
  leaderElect: true
schedulerName: my-custom-scheduler
```

Place the configuration file in `/etc/kubernetes/config/`.

#### Configuring the Custom Scheduler

Once the service and configuration files are set up:

1. **Start the scheduler**:
   ```bash
   systemctl daemon-reload
   systemctl start my-scheduler.service
   ```

2. **Enable it to start on boot**:
   ```bash
   systemctl enable my-scheduler.service
   ```

---

### Deploying the Scheduler as a Pod

Alternatively, you can deploy the custom scheduler as a pod within Kubernetes. This method uses a pod definition file and is useful for cloud environments or environments without direct access to systemd services.

#### Custom Scheduler Pod Configuration

Create a YAML manifest for the custom scheduler pod (`my-custom-scheduler.yaml`).

**Example Pod Manifest**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-custom-scheduler
  namespace: kube-system
spec:
  containers:
  - name: kube-scheduler
    image: k8s.gcr.io/kube-scheduler:v1.21.0
    command:
    - /usr/local/bin/kube-scheduler
    - --config=/etc/kubernetes/my-scheduler-config.yaml
    volumeMounts:
    - name: scheduler-config
      mountPath: /etc/kubernetes/my-scheduler-config.yaml
      subPath: my-scheduler-config.yaml
  volumes:
  - name: scheduler-config
    configMap:
      name: my-scheduler-config
```

Apply this YAML file to deploy the custom scheduler pod.

```bash
kubectl apply -f my-custom-scheduler.yaml
```

---

### Managing Multiple Schedulers in Kubernetes

When running multiple schedulers, Kubernetes allows you to configure each pod to be scheduled by a specific scheduler by specifying the `schedulerName` in the pod's manifest.

**Example Pod Manifest with Custom Scheduler**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
spec:
  schedulerName: my-custom-scheduler
  containers:
  - name: my-app-container
    image: nginx
```

This pod will now be scheduled by `my-custom-scheduler` instead of the default scheduler.

---

### Verifying the Custom Scheduler

After deploying the custom scheduler, you can verify its operation by checking the pods, events, and logs.

#### Checking Pods

To verify that the scheduler pod is running, use:

```bash
kubectl get pods --namespace kube-system
```

You should see the `my-custom-scheduler` pod listed.

#### Viewing Logs and Events

To view the events related to scheduling decisions made by your custom scheduler, run:

```bash
kubectl get events -o wide --namespace kube-system
```

Additionally, you can view the logs of the scheduler pod to inspect its behavior:

```bash
kubectl logs my-custom-scheduler --namespace kube-system
```

---

### Conclusion

Deploying and managing additional schedulers in Kubernetes allows for advanced scheduling configurations. Whether using systemd services or deploying the scheduler as a pod, configuring multiple schedulers provides flexibility for specialized workloads. Verifying the scheduler’s operation through `kubectl` commands ensures that it is functioning as expected.
