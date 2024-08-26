# Kubernetes Beginner to Pro

## Table of Contents

- Day 1: Introduction to Kubernetes
  - [YAML & Linting](docs/day1/yaml_and_linting.md)
  - [Linux Commands](docs/day1/linux_commands.md)
  - [Docker & Kubernetes](docs/day1/docker_and_kubernetes.md)


- Day 2: Kubernetes Architecture
  - [What are APIs](docs/day2/api.md)
  - [REST vs gRPC APIs](docs/day2/rest-grpc-api.md)
  - [Daemon vs DaemonSet](docs/day2/daemon-vs-daemonset.md)
  - [ETC vs ETCD](docs/day2/etc-vs-etcd.md)
  - [Kubernetes Architecture Overview](docs/day2/kubernetes-architecture-overview.md)
  - [Kube-Api-Server](docs/day2/kube-api-server.md)
  - [Admission Controllers](docs/day2/admission-controllers.md)
  - [ETCD](docs/day2/etcd.md)
  - [Kube-Scheduler](docs/day2/kube-scheduler.md)
  - [Kube-Controller-Manager](docs/day2/kube-controller-manager.md)
  - [Kube-Proxy and Kubelet](docs/day2/kube-proxy-and-kubelet.md)
  - [CRI, CSI, and CNI](docs/day2/cri-csi-cni.md)
  - [Kube-Proxy vs CNI](docs/day2/kube-proxy-vs-cni.md)
  - [Blog Summary: Scaling Kubernetes to 7,500 Nodes](docs/day2/scaling-kubernetes-to-7500-nodes-summary.md)
  - [Optional: Scenarios on Pod Communication](docs/day2/scenarios-on-pod-communication.md)
  - [Optional: Key Metrics](docs/day2/key-metrics.md)
  - [Resources](docs/day2/resources.md)

- Day 3: Cloud & on-Prem Clusters, Images, Debug Container & Pods, kubectl
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

- Day 4: Pods in Production, SideCar vs init Containers, Trivy, Kyverno
  - [Image Security: Trivy](docs/day4/image-security.md)
  - ['bat' Command](docs/day4/bat-command.md)
  - [Admission Control: Kyverno for Policy](docs/day4/kyverno.md)
  - [Kube Linter for Poduction Grade YAML](docs/day4/kube-linter.md)
  - [Kube Bench for Cluster Security](docs/day4/kube-bench.md)
  - [Static Pods, DaemonSets, Deployments, and Back-Off Algorithms](docs/day4/static-pods.md)
  - [Init Container](docs/day4/init-container.md)
  - [Sidecar vs Init Containers](docs/day4/sidecar-vs-init-containers.md)
  - [Pod/Container Lifecycle](docs/day4/container-cycle.md)
  - [Termination of Pods](docs/day4/termination-of-pods.md)
  - [Runtime Class](docs/day4/runtime-class.md)
    
- Day 5:  Labels, Selectors, ReplicaSets, Deployments, Production Strategies
  - [Labels & Selectors](docs/day5/labels-selectors.md)
  - [Revisiting Kubernetes Architeture](docs/day5/revisiting-kubernetes-architecture.md)
  - [Kubernetes YAML File Key Value Details](docs/day5/pod-yaml-file.md)
  - [See Labels](docs/day5/see-labels.md)
  - [kubectl Create vs Apply](docs/day5/kubectl-create-vs-apply.md)
  - [Update and Add Labels](docs/day5/update-and-add-labels.md)
  - [Metadata vs Spec Labels](docs/day5/metadata-vs-spec-labels.md)
  - [ReplicaSets](docs/day5/replicasets.md)
  - [Deployments](docs/day5/deployments.md)
  - [Deployment Strategies](docs/day5/deployment-strategy.md)
  - [Rollout Strategy](docs/day5/rollout-strategy.md)
 
- Day 6:  Services, Custom Domain Ingress
  - [Deployments and Service](docs/day6/problems-with-deployment.md)
  - [Application Deployment Begins](docs/day6/application-deployment-begins.md)
  - [Service Types](docs/day6/service-types.md)
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
  
- Day 7:  Adding SSL Cert to Applicaton, Cert-Manager, Gateway API, Namespaces, TLS
  - [Certificates Overview](docs/day7/certificates.md)
  - [Certificates: Issuer vs Cluster Issuer](docs/day7/certificates-issuer-vs-cluster-issuer.md)
  - [Public and Priavate Keys](docs/day7/public-and-private-keys.md)
  - [TLS and mTLS](docs/day7/tls-and-mtls.md)
  - [Let's Encrypt: Revolutionizing Web Security with Free, Automated SSL/TLS Certificates](docs/day7/lets-encrypt.md)
  - [Install Cert Manager](docs/day7/install-cert-manager.md)
















