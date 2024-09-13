### Liveness Probes in Kubernetes

#### Table of Contents (TOC)
1. [Introduction to Liveness Probes](#introduction)
2. [Types of Liveness Probes](#types-of-probes)
   - [1. HTTP Liveness Probe](#http-probe)
   - [2. Exec (Command) Liveness Probe](#exec-probe)
   - [3. TCP Socket Liveness Probe](#tcp-probe)
3. [Configuring Liveness Probes in YAML Files](#configuring-probes)
4. [Viewing Liveness Probe Status](#viewing-status)
5. [Practical Examples of Liveness Probes](#practical-examples)
   - [Example 1: HTTP Liveness Probe](#example-http)
   - [Example 2: Exec Liveness Probe](#example-exec)
   - [Example 3: TCP Liveness Probe](#example-tcp)
6. [Troubleshooting Liveness Probes](#troubleshooting)
7. [Conclusion](#conclusion)

---

### 1. Introduction to Liveness Probes <a name="introduction"></a>
**Liveness Probes** in Kubernetes ensure that containers inside a pod remain healthy by periodically checking their status. If a container is deemed unhealthy, Kubernetes will automatically restart it. This enables self-healing applications without the need for manual intervention.

Liveness probes are critical for ensuring application availability and can be configured using different methods, depending on the nature of the application.

[Back to TOC](#table-of-contents-toc)

---

### 2. Types of Liveness Probes <a name="types-of-probes"></a>

Liveness probes can be configured using three methods:

#### 2.1 HTTP Liveness Probe <a name="http-probe"></a>
- The probe sends an HTTP GET request to a specific path on the container. If the container responds with a successful status code (`2xx` or `3xx`), it is considered alive.

#### 2.2 Exec (Command) Liveness Probe <a name="exec-probe"></a>
- The probe executes a command inside the container. If the command exits with status `0`, the container is considered healthy. Any other exit code will mark the container as unhealthy.

#### 2.3 TCP Socket Liveness Probe <a name="tcp-probe"></a>
- The probe attempts to open a TCP connection to a specified port in the container. If the connection is successful, the container is considered healthy.

[Back to TOC](#table-of-contents-toc)

---

### 3. Configuring Liveness Probes in YAML Files <a name="configuring-probes"></a>

Liveness probes are defined within the `livenessProbe` field under a container specification in the pod’s YAML file. They can be configured for **HTTP GET**, **Exec**, or **TCP Socket** probes. Let's go over how to configure each type:

#### 3.1 HTTP Liveness Probe Example
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: http-liveness-pod
spec:
  containers:
    - name: main-app
      image: my-app-image
      livenessProbe:
        httpGet:
          path: /healthz        # This endpoint should return 2xx/3xx for healthy state
          port: 8080            # Port exposed by the app container
        initialDelaySeconds: 5   # Wait 5 seconds before the first probe
        periodSeconds: 10        # Check every 10 seconds
```
In this example, the probe sends a GET request to `/healthz` on port `8080`. If the response is healthy, the container is fine; otherwise, it will be restarted.

#### 3.2 Exec Liveness Probe Example
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exec-liveness-pod
spec:
  containers:
    - name: main-app
      image: my-app-image
      livenessProbe:
        exec:
          command:
            - cat
            - /tmp/healthy       # The file /tmp/healthy should exist for the container to be healthy
        initialDelaySeconds: 5    # Wait 5 seconds before the first probe
        periodSeconds: 10         # Check every 10 seconds
```
Here, the probe runs the `cat /tmp/healthy` command inside the container. If the file `/tmp/healthy` exists and is readable, the container is considered healthy.

#### 3.3 TCP Liveness Probe Example
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tcp-liveness-pod
spec:
  containers:
    - name: main-app
      image: my-app-image
      livenessProbe:
        tcpSocket:
          port: 8080             # Kubernetes will attempt to connect to this port
        initialDelaySeconds: 5    # Wait 5 seconds before the first probe
        periodSeconds: 10         # Check every 10 seconds
```
This probe tries to establish a TCP connection to port `8080`. If successful, the container is marked as healthy.

[Back to TOC](#table-of-contents-toc)

---

### 4. Viewing Liveness Probe Status <a name="viewing-status"></a>

To check the status of liveness probes and whether they are causing container restarts, you can use the following commands:

#### 4.1 Get Pod Details
```bash
kubectl describe pod <pod-name>
```
This will show detailed information about the pod, including the status of liveness probes and whether they have failed.

#### 4.2 Check Pod Events
```bash
kubectl get events --field-selector involvedObject.name=<pod-name>
```
This will display events related to the pod, such as any liveness probe failures and subsequent container restarts.

#### 4.3 View Logs of the Container
```bash
kubectl logs <pod-name> -c <container-name>
```
Use this to inspect container logs and gather more information about what may be causing the probe to fail.

[Back to TOC](#table-of-contents-toc)

---

### 5. Practical Examples of Liveness Probes <a name="practical-examples"></a>

Let's look at how these probes work in a real scenario:

#### 5.1 Example 1: HTTP Liveness Probe <a name="example-http"></a>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: http-liveness-pod
spec:
  containers:
    - name: main-app
      image: my-app-image
      livenessProbe:
        httpGet:
          path: /healthz           # Kubernetes checks this endpoint for health status
          port: 8080               # The port the application is listening on
        initialDelaySeconds: 5      # Wait 5 seconds after starting the container
        periodSeconds: 10           # Probe every 10 seconds
```
**Output:**
If the `/healthz` endpoint returns a `200 OK` response, the container is marked as healthy. Otherwise, Kubernetes restarts the container.

[Back to TOC](#table-of-contents-toc)

---

#### 5.2 Example 2: Exec Liveness Probe <a name="example-exec"></a>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exec-liveness-pod
spec:
  containers:
    - name: main-app
      image: my-app-image
      livenessProbe:
        exec:
          command:
            - cat
            - /tmp/healthy         # The presence of /tmp/healthy signals the container is alive
        initialDelaySeconds: 5
        periodSeconds: 10
```
**Output:**
If the file `/tmp/healthy` exists, the container is healthy. If the file is missing or unreadable, the container will be restarted.

[Back to TOC](#table-of-contents-toc)

---

#### 5.3 Example 3: TCP Liveness Probe <a name="example-tcp"></a>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tcp-liveness-pod
spec:
  containers:
    - name: main-app
      image: my-app-image
      livenessProbe:
        tcpSocket:
          port: 8080               # Probe will try to establish a TCP connection to this port
        initialDelaySeconds: 5
        periodSeconds: 10
```
**Output:**
If Kubernetes can establish a TCP connection to port `8080`, the container is healthy. If not, the container will be restarted.

[Back to TOC](#table-of-contents-toc)

---

### 6. Troubleshooting Liveness Probes <a name="troubleshooting"></a>

Here are some common troubleshooting tips for liveness probes:

#### 6.1 Increase the `initialDelaySeconds`
If your container takes a while to become fully operational (e.g., it initializes resources), you can increase the `initialDelaySeconds` to prevent Kubernetes from prematurely marking it as unhealthy.

#### 6.2 Check Probe Path and Port
Ensure that the `path` in an HTTP probe or the `port` in a TCP probe is correct. For example, verify that the `/healthz` endpoint is correctly implemented in the application.

#### 6.3 Review Probe Timing
Adjust `periodSeconds` and `timeoutSeconds` so the probe fits the operational needs of the application. A too-aggressive probe might cause false positives.

#### 6.4 View Events and Logs
Check the pod’s events and logs (using `kubectl get events` or `kubectl logs`) to diagnose why a probe is failing.

[Back to TOC](#table-of-contents-toc)

---

### 7.

 Conclusion <a name="conclusion"></a>

Liveness probes in Kubernetes ensure that applications stay healthy by automatically detecting and recovering from failures. By configuring HTTP, Exec, or TCP probes, you can manage container restarts and keep your application running smoothly. Understanding how to define and troubleshoot these probes is key to maintaining application availability in a Kubernetes cluster.

[Back to TOC](#table-of-contents-toc)

---

This concludes the tutorial on liveness probes in Kubernetes. If you'd like to dive deeper into any of these probe types or see additional examples, feel free to explore further!
