# **Kubernetes Gateway API on GCP, AWS, and Azure**

This tutorial guides you through setting up the Kubernetes Gateway API on Google Cloud Platform (GCP), Amazon Web Services (AWS), and Microsoft Azure. The tutorial is structured to prioritize GCP details, with AWS and Azure combined where there is overlap. Comments are used in the YAML manifests to highlight cross-cloud compatibility, minimizing the need for multiple YAML files.

---

## **Table of Contents**

- [Part 1: Google Cloud Platform (GCP)](#part-1-google-cloud-platform-gcp)
  - [Enabling Gateway API on GCP](#enabling-gateway-api-on-gcp)
  - [Listing Gateway Classes](#listing-gateway-classes)
  - [Creating a Gateway on GCP](#creating-a-gateway-on-gcp)
- [Part 2: Amazon Web Services (AWS) and Microsoft Azure (Azure)](#part-2-amazon-web-services-aws-and-microsoft-azure-azure)
  - [Creating a Gateway Class on AWS and Azure](#creating-a-gateway-class-on-aws-and-azure)
  - [Listing Gateway Classes](#listing-gateway-classes-1)
  - [Creating a Gateway on AWS and Azure](#creating-a-gateway-on-aws-and-azure)
- [Summary](#summary)
- [References](#references)

---

## **Part 1: Google Cloud Platform (GCP)**

### **Enabling Gateway API on GCP**

To enable the Gateway API on GCP, you need to update your Kubernetes cluster using the `gcloud` command. This command automatically creates Gateway Classes for you.

#### **Command:**
```bash
gcloud container clusters update day7-kubernetes-hindi \
    --location=us-central1-c \
    --gateway-api-standard
```

This command will update your GKE cluster and create standard Gateway Classes that can be used to set up Gateways.

[Back to TOC](#table-of-contents)

### **Listing Gateway Classes**

After updating your cluster, you can list the available Gateway Classes:

#### **Command:**
```bash
kubectl get gatewayclasses
```

#### **Sample Output:**
```plaintext
NAME                      CONTROLLER                      AGE
gke-l7-global-gateway     networking.gke.io/global        5m
gke-l7-regional-gateway   networking.gke.io/regional      5m
```

[Back to TOC](#table-of-contents)

### **Creating a Gateway on GCP**

Here’s an example of creating a Gateway using one of the pre-configured Gateway Classes:

#### **Sample YAML for GCP Gateway:**
```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: Gateway
metadata:
  name: my-gcp-gateway
  namespace: default
spec:
  gatewayClassName: gke-l7-regional-gateway
  listeners:
    - name: http
      protocol: HTTP
      port: 80
```

#### **Applying the YAML:**

```bash
kubectl apply -f my-gcp-gateway.yaml
```

#### **Verifying the Gateway:**

```bash
kubectl get gateways
```

#### **Sample Output:**
```plaintext
NAME             CLASS                       AGE
my-gcp-gateway   gke-l7-regional-gateway     1m
```

[Back to TOC](#table-of-contents)

---

## **Part 2: Amazon Web Services (AWS) and Microsoft Azure (Azure)**

AWS and Azure share some similarities in how you define and manage Gateway Classes and Gateways. Below, we outline the steps for setting up the Gateway API on both platforms, with comments highlighting where the configuration can be shared.

### **Creating a Gateway Class on AWS and Azure**

Both AWS and Azure require you to manually define Gateway Classes.

#### **Sample YAML for Gateway Class (Shared):**
```yaml
apiVersion: gateway.networking.k8s.io/v1alpha2
kind: GatewayClass
metadata:
  name: cloud-nlb  # This name can be reused on both AWS and Azure
spec:
  controllerName: eks.amazonaws.com/aws-network-load-balancer  # AWS NLB controller
  # controllerName: azure.com/application-gateway              # Uncomment for Azure Application Gateway
```

- **ControllerName:** 
  - For AWS, use `eks.amazonaws.com/aws-network-load-balancer` to leverage the AWS Network Load Balancer.
  - For Azure, use `azure.com/application-gateway` to leverage the Azure Application Gateway.

#### **Applying the Gateway Class:**

```bash
kubectl apply -f cloud-gatewayclass.yaml
```

[Back to TOC](#table-of-contents)

### **Listing Gateway Classes**

You can list the available Gateway Classes to verify:

#### **Command:**
```bash
kubectl get gatewayclasses
```

#### **Sample Output:**
```plaintext
NAME        CONTROLLER                                AGE
cloud-nlb   eks.amazonaws.com/aws-network-load-balancer   2m
# Or for Azure
# cloud-nlb   azure.com/application-gateway             2m
```

[Back to TOC](#table-of-contents)

### **Creating a Gateway on AWS and Azure**

Once your Gateway Class is set up, you can create a Gateway using the following YAML. The same YAML can be used across both AWS and Azure by simply choosing the appropriate controller when applying the Gateway Class.

#### **Sample YAML for Gateway (Shared):**
```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: Gateway
metadata:
  name: my-cloud-gateway
  namespace: default
spec:
  gatewayClassName: cloud-nlb  # Shared name for both AWS and Azure
  listeners:
    - name: http
      protocol: HTTP
      port: 80
```

#### **Applying the Gateway YAML:**

```bash
kubectl apply -f my-cloud-gateway.yaml
```

#### **Verifying the Gateway:**

```bash
kubectl get gateways
```

#### **Sample Output:**
```plaintext
NAME             CLASS       AGE
my-cloud-gateway cloud-nlb   1m
```

[Back to TOC](#table-of-contents)

---

## **Summary**

- **GCP:** Automatically creates Gateway Classes when the Gateway API is enabled. You can directly create and apply Gateways using these pre-configured classes.
- **AWS and Azure:** Require manual creation of Gateway Classes. You can reuse the same Gateway Class and Gateway YAML files across both platforms by choosing the appropriate controller for each cloud provider.

This approach streamlines your Kubernetes networking setup across multiple cloud providers, ensuring consistent traffic management regardless of the underlying infrastructure.

[Back to TOC](#table-of-contents)


## **References:**
[GCP Documentation for Gateway API](https://cloud.google.com/kubernetes-engine/docs/concepts/gateway-api)
