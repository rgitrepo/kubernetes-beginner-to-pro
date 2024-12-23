
# Kubernetes Beginner to Pro

## Table of Contents
<details>
<summary>Day 1: Introduction to Kubernetes</summary>

- [YAML & Linting](docs/day1/yaml_and_linting.md)
- [Linux Commands](docs/day1/linux_commands.md)
- [Docker & Kubernetes](docs/day1/docker_and_kubernetes.md)

</details>

<details>
<summary>Day 2: Kubernetes Architecture</summary>

- [What are APIs](docs/day2/api.md)
- [REST vs gRPC APIs](docs/day2/rest-grpc-api.md)
- [DaemonSet, Daemon, and ReplicaSet](docs/day2/daemonset-daemon-and-replicaset.md)
- [ETC vs ETCD](docs/day2/etc-vs-etcd.md)
- [Kubernetes Architecture Overview](docs/day2/kubernetes-architecture-overview.md)
- [Kube-Api-Server](docs/day2/kube-api-server.md)
- [Admission Controllers](docs/day2/admission-controllers.md)
- [ETCD](docs/day2/etcd.md)
- [ETCD Topology: Stacked with External](docs/day2/etcd-topology-stacked-vs-external.md)
- [Kube-Scheduler](docs/day2/kube-scheduler.md)
- [Kube-Controller-Manager](docs/day2/kube-controller-manager.md)
- [Kube-Proxy and Kubelet](docs/day2/kube-proxy-and-kubelet.md)
- [CRI, CSI, and CNI](docs/day2/cri-csi-cni.md)
- [Kube-Proxy vs CNI](docs/day2/kube-proxy-vs-cni.md)
- [Blog Summary: Scaling Kubernetes to 7,500 Nodes](docs/day2/scaling-kubernetes-to-7500-nodes-summary.md)
- [Optional: Scenarios on Pod Communication](docs/day2/scenarios-on-pod-communication.md)
- [Resources](docs/day2/resources.md)

</details>

<details>
<summary>Day 3: Cloud & on-Prem Clusters, Images, Debug Container & Pods, kubectl</summary>

- [Kubernetes Clusters - GKE](docs/day3/kubernetes-clusters.md)
- [Kind Cluster](docs/day3/kind-cluster.md)
- [Interview Questions: Kubernetes Version Management](docs/day3/interview-questions.md)
- [Kubeadm](docs/day3/kubeadm.md)
- [Cluster & Objects](docs/day3/cluster-and-objects.md)
- [Images](docs/day3/docker-containers.md)
- [Containers](docs/day3/understanding-containers.md)
- [Kubectl](docs/day3/kubectl.md)
- [Pods](docs/day3/pods.md)
- [Exec In The Pod](docs/day3/exec-in-the-pod.md)
- [Debug Containers](docs/day3/debug-containers.md)

</details>

<details>
<summary>Day 4: Static Pods, Pod & Container Lifecycle, SideCar vs init Containers, Trivy, Kyverno, Kube Linter, Kube Bench, Backoff Algorithms</summary>

- [Trivy: Image Security](docs/day4/image-security.md)
- ['bat' Command](docs/day4/bat-command.md)
- [Policy: Kyverno (Admission Controller)](docs/day4/kyverno.md)
- [Kube Linter: Poduction Grade YAML](docs/day4/kube-linter.md)
- [Kube Bench: Cluster Security](docs/day4/kube-bench.md)
- [Static Pods](docs/day4/static-pods.md)
- [Backoff Algorithms](docs/day4/backoff-algorithms.md)
- [Init Containers](docs/day4/init-container.md)
- [Sidecar vs Init Containers](docs/day4/sidecar-vs-init-containers.md)
- [Overriding CMD and ENTRYPOINT of Dockerfile in Pod Definition YAML](docs/day4/overriding-command-and-entrypoint-in-kubernetes-pod-defiintion-yaml.md)
- [Pod and Container Lifecycle](docs/day4/container-cycle.md)
- [Termination of Pods](docs/day4/termination-of-pods.md)
- [Runtime Class](docs/day4/runtime-class.md)
  
</details>

<details>
<summary>Day 5: Labels, Selectors, ReplicaSets, Deployments, Deployment Strategies, Updates, Production Strategies</summary>

- [Labels & Selectors Using Yaml](docs/day5/labels-selectors.md)
- [Labels Using Command Line](docs/day5/update-and-add-labels.md)
- [Revisiting Kubernetes Architecture](docs/day5/revisiting-kubernetes-architecture.md)
- [Kubernetes YAML File Key Value Details](docs/day5/pod-yaml-file.md)
- [Commands to see Labels](docs/day5/see-labels.md)
- [kubectl Create vs Apply](docs/day5/kubectl-create-vs-apply.md)
- [Metadata vs Spec Labels](docs/day5/metadata-vs-spec-labels.md)
- [ReplicaSets](docs/day5/replicasets.md)
- [Deployments](docs/day5/deployments.md)
- [Labels and Selectors: Comparing Service, ReplicatSet, and Deployment](docs/day5/labelsl-and-selectors-comparing-service-replicaset-and-deployment.md)
- [Pod's `ownerReference` field](docs/day5/pods-ownerrefrence-field.md)
- [`kubectl explain` command](docs/day5/explain-command.md)
- [Deployment Strategies](docs/day5/deployment-strategy.md)
- [Rollout Strategy](docs/day5/rollout-strategy.md)

</details>

<details>
<summary>Day 6: Services, Custom Domain Ingress</summary>

- [Deployments and Service](docs/day6/problems-with-deployment.md)
- [Application Deployment Begins](docs/day6/application-deployment-begins.md)
- [Service Types: Cluster IP, Node Port, Load Balancer, Headless, External Name](docs/day6/service-types.md)
- [Endpoints vs Endpoint-Slices](docs/day6/endpoints-and-endpoint-slices.md)
- [Editing Endpoint Slices](docs/day6/editing-endpoint-slices.md)
- [Endpoint Slices Review](docs/day6/endpoint-slices-review.md)
- [Configure the --max-endpoints-per-slice Flag](docs/day6/configure-the-max-endpoints-per-slice-flag.md)
- [Ports in Services](docs/day6/ports-in-services.md)
- [IPs: Cluster, Node, and Service](docs/day6/kubernetes-ips.md)
- [ClusterIP (Service IP) vs Service IP](docs/day6/clusterip-vs-service-ip.md)
- [Load Balancer Service](docs/day6/load-balancer-service.md)
- [Ingress](docs/day6/ingress-and-ingress-controller.md)
- [Names of Some Ingress Controllers](docs/day6/names-of-some-ingress-controllers.md)
- [Annotations](docs/day6/annotations.md)
- [Ingress Class](docs/day6/use-of-ingress-class.md)
- [DNS Record Types: CNAME vs A-Record](docs/day6/cname-vs-a-record.md)
- [Optional Project: Customizing Load Balancer Algorithms in Kubernetes](docs/day6/customizing-load-balancer-algorithms.md)
- [Optional Project: Geolocation based Ingress Project](docs/day6/geolocation-ingress-step-by-step-guide.md)

</details>

<details>
<summary>Day 7: SSL, TLS, Cert-Manager, Gateway API, Namespaces, Context, Resource Management</summary>

- [Certificates Overview](docs/day7/certificates.md)
- [Certificates: Issuer vs Cluster Issuer](docs/day7/certificates-issuer-vs-cluster-issuer.md)
- [Public and Private Keys](docs/day7/public-and-private-keys.md)
- [TLS and mTLS](docs/day7/tls-and-mtls.md)
- [Let's Encrypt: Certificiation Authority with Free, Automated SSL/TLS Certificates](docs/day7/lets-encrypt.md)
- [Install Cert Manager](docs/day7/install-cert-manager.md)
- [Cluster Issuer YAML Manifest](docs/day7/cluster-issuer-yaml-manifest.md)
- [Prod vs Staging Servers](docs/day7/prod-vs-staging-server.md)
- [Adding Certificates](docs/day7/adding-certificates.md)
- [Project TBD: Currency Converter + Weather App + Best Places to Visit](docs/day7/weather-app.md)
- [Problems with Ingress](docs/day7/problems-with-ingress.md)
- [Gateway API Advantages](docs/day7/advantages-of-gateway-api.md)
- [Gateway API Configuration](docs/day7/gateway-api.md)
- [HTTPRoute - forwardTo vs backendRefs](docs/day7/forwardto-vs-backendrefs.md)
- [Gateway API on Cloud Providers: GCP, AWS, and Azure](docs/day7/gateway-api-on-gcp-aws-and-azure.md)
- [Applying Multiple YAML files](docs/day7/applying-multiple-yaml-files.md)
- [Namespace](docs/day7/namespace.md)
- [Context & Namespace](docs/day7/context.md)
- [Noisy Neighbour Problem](docs/day7/noisy-neighbour-problem.md)
- [Resource Management Commands](docs/day7/resource-management.md)
  

</details>

<details>
<summary>Day 8: Scheduling, Taints & Tolerations, Node Affinity, QoS, OpenCost, Limit Ranges, Resource Quotas, Memory & CPU Units</summary>

- [CPU vs Memory](docs/day8/cpu-vs-memory.md)
- [OOM Kill - Out Of Memory Kill](docs/day8/oom-kill.md)
- [QoS: Quality of Service](docs/day8/qos-quality-of-service.md)
- [Policies: API Objects, Contollers, Kubelet ](docs/day8/policies.md)
- [Memory & CPU Units](docs/day8/memory-and-cpu-units.md)
- [Pods, Resource Quotas, and LimitRanges](docs/day8/limitranges-and-resourcequotas.md)
- [Kyverno vs. LimitRanges and ResourceQuotas](docs/day8/kyverno-vs-limitranges-and-resourcequotas.md)
- [Cost Optimization with OpenCost](docs/day8/cost-optimization-with-opencost.md)
- [Taints & Tolerations](docs/day8/taints-and-tolerations.md)
- [Scheduler Health](docs/day8/scheduler-health.md)
- [Scheduler Bypass: Nodename & NodeSelector](docs/day8/nodename-and-nodeselector.md)
- [Scheduler Bypass: NodeName vs Binding](docs/day8/nodename-vs-binding.md)
- [Deploy Additional Scheduler](docs/day8/deploy-additional-scheduler.md)
- [Pod Topology Spread Constraints](docs/day8/pod-topology-spread-constraints.md)
- [Node Affinity and Pod Affinity/Anti-Affinity](docs/day8/node-affinity-and-pod-affinity-anti-affinity.md)
- [Taints & Tolerations vs Node Affinity](docs/day8/taints-and-tolerations-vs-node-affinity.md)

</details>

<details>
<summary>Day 9: Data Storage & User Management in Kubernetes, RBAC, Volumes, Storage Classes</summary>

- [Authentication and Authorization (AuthN & AuthZ)](docs/day9/authentication-and-authorization.md)
- [Kube Config File](docs/day9/kubeconfig-file.md)
- [Questions & Answers - Cluster](docs/day9/cluster-questions-and-answers.md)
- [RBAC: Role-Based Access Control, Role, RoleBindings, ClusterRole, ClusterRoleBindings](docs/day9/rbac.md)
- [In-Tree vs Out-of-Tree](docs/day9/in-tree-vs-out-of-tree.md)
- [Volumes & Dynamic Provisioning](docs/day9/volumes-and-dynamic-provisioning.md)
- [PV & PVC: Persistent Volumes & Persistent Volume Claims](docs/day9/persisten-volumes-and-persistent-volume-claims.md)
- [PV Modes, Dynamic Provisioning & Reclaim Policies](docs/day9/persistent-volume-modes-and-dynamic-provisioning.md)
- [Storing Images, Videos, and Other Files in Kubernetes Volumes](docs/day9/storing-images-videos-and-other-files-in-volumes.md)
- [Statically and Dynamically Provisioned PVCs](docs/day9/statically-and-dynamically-provisioned-pvc.md)
- [Role of StorageClasses in Static Provisioning of Persistent Volumes](docs/day9/role-of-storageclass-in-static-provisioning-of-persistent-volumes.md)
- [Role of StorageClasses in Dynamic Provisioning of Persistent Volumes](docs/day9/role-of-storageclass-in-dynamic-provisioning-of-persistent-volumes.md)
- [Reasons for PVC Pending State](docs/day9/reasons-for-pvc-pending-state.md)
- [Mastering kubectl Commands for PV, PVC, and SC](docs/day9/mastering-kubectl-commands-for-pv-pvc-and-sc.md)
- [Why Many Companies Avoid Running Databases on Kubernetes](docs/day9/why-many-companies-avoid-running-databases-on-kubernetes.md)

  


</details>


<details>
<summary>Day 10: Service Mesh, Secrets, CMs, External Secret Operator, Google Cloud Secret Manager</summary>

- [Kubernetes `env` Value Types](docs/day10/env-value-types.md)
- [ConfigMaps - Introduction & Overview](docs/day10/configmaps.md)
- [ConfigMaps - Consuming via Environment Variables](docs/day10/consuming-configmaps-using-environment-variables.md)
- [ConfigMaps - Consuming in Pods](docs/day10/consuming-configmaps-from-pod-commands.md)
- [ConfigMaps - Consuming via Files](docs/day10/configmaps-creation-and-consumption-from-files.md)
- [ConfigMaps - Consuming via Directories](docs/day10/consuming-configmaps-from-directories.md)
- [ConfigMaps - RBAC with Cloud-Based Examples](docs/day10/rbac-with-cloud-based-examples.md)
- [ConfigMaps - Interview Questions](docs/day10/interview-questions-configmaps.md)
- [Secrets - Introduction & Overview](docs/day10/secrets.md)
- [Secrets - Base64 Encoding & Decoding](docs/day10/secrets-base64-encoding-and-decoding.md)
- [Secrets - Creating Kubernetes Secrets](docs/day10/secrets-creating-kubernetes-secrets.md)
- [Secrets - Consuming via Environment Variables](docs/day10/secrets-consuming-via-environment-variables.md)
- [Secrets - Consuming via Files (Volume Mounts)](docs/day10/secrets-consuming-via-files.md)
- [Secrets - Consuming via Directories (Volume Mounts)](docs/day10/secrets-consuming-via-directories.md)
- [Secrets - Consuming in Pods](docs/day10/secrets-consuming-in-pods.md)
- [Secrets - How Pods Pull Images from a Private Docker Registry Using `ImagePullSecrets`](docs/day10/how-pods-pull-images-from-a-private-docker-registry-using-imagepullsecrets.md)
- [Secrets - Interview Questions](docs/day10/interview-questions-secrets.md)
- [Sealed Secrets](docs/day10/sealed-secrets.md)
- [External Secret Operator & GCSM](docs/day10/external-secret-operator-and-gcsm.md)
- [kubectl Commands to Manage and Troubleshoot Secrets](docs/day10/kubectl-commands-to-manage-and-troubleshoot-secrets.md)
- [Implementing TLS, mTLS, and Service Mesh](docs/day10/implementing-tls-mtls-and-service-mesh.md)
- [OSI Model](docs/day10/osi-model.md)
- [Sidecar vs Sidecarless Model in Service Mesh](docs/day10/sidecar-vs-sidecarless-pattern.md)
- [Distinguishing Sidecar Containers from Regular Containers](docs/day10/distinguishing-sidecar-containers-from-regular-containers.md)
- [eBPF (extended Berkeley Packet Filter)](docs/day10/ebpf.md)
- [Cilium](docs/day10/cilium.md)
- [Istio](docs/day10/istio.md)
- [Istio - ztunnel](docs/day10/ztunnel.md)
- [Istio - Dashboard](docs/day10/istio-dashboard.md)
- [Liveness, Readiness, and Startup Probes](docs/day10/liveness-readiness-and-startup-probes.md)
- [Liveness Probes](docs/day10/liveness-probes.md)
- [Readiness Probes](docs/day10/readiness-probes.md)
- [Startup Probes](docs/day10/startup-probes.md)

  
</details>

<details>
<summary>Day 11: Monitoring, Logging, Tracing, Profiling, Prometheus, Jaeger, OTel, Grafana, eBPF</summary>


- [Monitoring Resources in Kubernetes using `kubectl top`](docs/day11/monitoring-resources-in-kubernetes-using-kubectl-top.md)
- [How to Scrape Metrics](docs/day11/how-to-scrape-metrics.md)
- [Key Metrics](docs/day11/key-metrics.md)
- [Prometheus](docs/day11/prometheus.md)
- [Grafana](docs/day11/grafana.md)
- [Logging](docs/day11/logging.md)
- [Tracing & Profiling](docs/day11/tracing-and-profiling.md)
- [eBPF-based Tools (Pixie) & OpenTelemetry (OTel)](docs/day11/ebpf-tools-and-otel.md)
- [Questions & Answers](docs/day11/questions-and-answers.md)

</details>

<details>
<summary>Day 12: Runtime Security & HA Config, Network Policies, Falco, OPA, Tracee, Security Context</summary>

- [HA Clusters](docs/day12/ha-clusters.md)
- [Interview Questions](docs/day12/interview-questions.md)
- [Benchmarking with `kube-bench`](docs/day12/benchmarking-with-kube-bench.md)
- [Upgrading & Downgrading Clusters: Drain, Cordon, Uncordon](docs/day12/upgrading-and-downgrading-clusters.md)
- [Runtime Security Intro](docs/day12/runtime-security-intro.md)
- [Security Context & Capabilities](docs/day12/security-context-and-capabilities.md)
- [Seccomp](docs/day12/seccomp.md)
- [Tracee](docs/day12/tracee.md)
- [Falco & OPA](docs/day12/falco-and-opa.md)
- [Network Policies](docs/day12/network-policies.md)
- [kubectx & kubens: Switchin Between Context & Namespaces](docs/day12/kubectx-and-kubens.md)

</details>

<details>
<summary>Day 13: Ad-Hoc Topics </summary>

- [Kubeadm vs Manual Installation](docs/day13/kubeadm-vs-manual-installation.md)
- [Seeing logs with `journalctl` ](docs/day13/seeing-logs-with-journalctl.md)
- [Weave Net Install for CNI](docs/day13/weavenet-install-for-cni.md)


</details>

