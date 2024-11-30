
## Network Plugin in Kubernetes


There are several plugins available, and these are some.

### 1. Weave Net:

To install,

```
kubectl apply -f
```

https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

You can find details about the network plugins in the following documentation :

https://kubernetes.io/docs/concepts/cluster-administration/addons/#networking-and-network-policy

### 2. Flannel :

To install,

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml
```

Note: As of now, flannel does not support Kubernetes network policies.

3. Calico :

### To install,

```
curl https://docs.projectcalico.org/manifests/calico.yaml -O
```

Apply the manifest using the following command.

```
kubectl apply -f calico.yaml
```

Calico is said to have the most advanced cni network plugin.

In the CKA and CKAD exams, you won’t be asked to install the cni plugin. But if asked, you will be provided with the exact URL to install it.

**Note: If there are multiple CNI configuration files in the directory, the kubelet uses the configuration file that comes first by name in lexicographic order.**


The `-` at the end of the command is significant because it instructs `kubectl` to read input from **standard input (stdin)** rather than expecting a file. Here's why it's necessary in this context:

### Command Breakdown:
1. **Without the final `-`:**
   ```bash
   curl -L https://github.com/weaveworks/weave/releases/download/latest_release/weave-daemonset-k8s-1.11.yaml | kubectl apply -f
   ```
   - `kubectl apply -f` expects a file path or URL as the next argument. Since no file path or URL is provided after `-f`, the command fails with an error like:
     ```
     error: --filename (-f) is required
     ```

2. **With the final `-`:**
   ```bash
   curl -L https://github.com/weaveworks/weave/releases/download/latest_release/weave-daemonset-k8s-1.11.yaml | kubectl apply -f -
   ```
   - The `-` tells `kubectl` to accept the input directly from the pipe (stdin), where the YAML content streamed by `curl` is being passed.
   - `kubectl apply -f -` effectively processes the YAML content coming from `curl` as if it were reading it from a file.

### Key Role of `-`:
- It acts as a placeholder indicating that `kubectl` should process the input directly from the output of `curl`, enabling commands like these to work seamlessly in a pipeline.
- Without the `-`, `kubectl` doesn't know what input to use for the `-f` flag, leading to a failure.

### Why Does the Pipe Require `-`?
Pipelines (`|`) pass data from the stdout of one command (`curl`) to the stdin of another (`kubectl`). The `-` in this case explicitly tells `kubectl` to read its input from stdin, which is necessary when chaining commands like this.

---

### Additional Notes:
- If the YAML file was downloaded locally, you could avoid using `-` by providing the file path:
  ```bash
  curl -L -o weave-daemonset.yaml https://github.com/weaveworks/weave/releases/download/latest_release/weave-daemonset-k8s-1.11.yaml
  kubectl apply -f weave-daemonset.yaml
  ```
- Alternatively, you could pass the URL directly to `kubectl` if supported, eliminating the need for `curl`:
  ```bash
  kubectl apply -f https://github.com/weaveworks/weave/releases/download/latest_release/weave-daemonset-k8s-1.11.yaml
  ```
