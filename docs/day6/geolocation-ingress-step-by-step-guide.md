### Geolocation-Based Ingress in Kubernetes: A Step-by-Step Tutorial

### Table of Contents (TOC)

1. [Introduction](#introduction)
2. [Step 1: Create Hosted Zones in Route 53](#step-1-create-hosted-zones-in-route-53)
3. [Step 2: Set Up GeoDNS Records in Route 53](#step-2-set-up-geodns-records-in-route-53)
4. [Step 3: Configure ExternalDNS in Kubernetes](#step-3-configure-externaldns-in-kubernetes)
   - [Create IAM Policy for ExternalDNS](#create-iam-policy-for-externaldns)
   - [Create IAM Role and Attach Policy](#create-iam-role-and-attach-policy)
   - [Install ExternalDNS in Your Clusters](#install-externaldns-in-your-clusters)
5. [Step 4: Integrate GeoIP with Route 53](#step-4-integrate-geoip-with-route-53)
6. [Step 5: Create Ingress Resources in Kubernetes](#step-5-create-ingress-resources-in-kubernetes)
7. [Step 6: Test the Geolocation-Based Routing](#step-6-test-the-geolocation-based-routing)
   - [Verify DNS Records](#verify-dns-records)
   - [Access the Application](#access-the-application)
8. [Conclusion](#conclusion)

---

### Introduction

Geolocation-based ingress in Kubernetes allows traffic to be directed to specific backend services based on the geographical location of the client making the request. This is useful for improving performance by routing users to the nearest data center, enhancing compliance by adhering to region-specific regulations, and optimizing resource utilization by distributing load based on location.

[Back to TOC](#table-of-contents-toc)

---

### Step 1: Create Hosted Zones in Route 53

1. **Log in to the AWS Management Console** and navigate to **Route 53**.
2. **Create a Hosted Zone**:
   - Go to **Hosted Zones** and click **Create Hosted Zone**.
   - Enter your domain name (e.g., `myapp.com`) and leave the type as **Public Hosted Zone**.
   - Click **Create Hosted Zone**.

   **Output**:
   You should now see your domain listed in the Hosted Zones with a unique **Hosted Zone ID**.

[Back to TOC](#table-of-contents-toc)

---

### Step 2: Set Up GeoDNS Records in Route 53

1. **Create Geolocation Routing Policies**:
   - Inside your Hosted Zone, click **Create Record**.
   - Choose **Geolocation** for the routing policy.
   - **Create Records** for each region:
     - **North America**: Direct traffic to `us-west-1`.
     - **Europe**: Direct traffic to `eu-central-1`.
     - **Default**: Add a fallback region (e.g., `us-west-1`) for all other locations.

   **Output**:
   After creation, Route 53 should show records with geolocation routing policies tied to different regions.

[Back to TOC](#table-of-contents-toc)

---

### Step 3: Configure ExternalDNS in Kubernetes

#### Create IAM Policy for ExternalDNS

1. **Create IAM Policy**:
   - Go to **IAM** in AWS Management Console.
   - Create a new **IAM Policy** with the following permissions:

   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": [
                   "route53:ChangeResourceRecordSets",
                   "route53:ListHostedZones",
                   "route53:ListResourceRecordSets"
               ],
               "Resource": ["arn:aws:route53:::hostedzone/<your-hosted-zone-id>"]
           }
       ]
   }
   ```

   Replace `<your-hosted-zone-id>` with your actual Hosted Zone ID.

[Back to TOC](#table-of-contents-toc)

---

#### Create IAM Role and Attach Policy

1. **Create IAM Role**:
   - Create an IAM Role that Kubernetes can assume and attach the IAM policy you just created.

[Back to TOC](#table-of-contents-toc)

---

#### Install ExternalDNS in Your Clusters

1. **Install ExternalDNS** in your Kubernetes clusters (`us-west-1` and `eu-central-1`):
   - Create a `values.yaml` file for the ExternalDNS Helm chart with the following content:

   ```yaml
   provider: aws
   aws:
     region: us-west-1  # Adjust based on your cluster's region
     zoneType: public
   domainFilters:
     - myapp.com
   policy: upsert-only
   txtOwnerId: "externaldns"
   rbac:
     create: true
   serviceAccount:
     create: true
   ```

   - Install ExternalDNS with Helm:

   ```bash
   helm repo add bitnami https://charts.bitnami.com/bitnami
   helm install externaldns bitnami/external-dns -f values.yaml
   ```

   **Output**:
   ExternalDNS should now be running in your cluster, automatically creating DNS records in Route 53 based on your Ingress resources.

[Back to TOC](#table-of-contents-toc)

---

### Step 4: Integrate GeoIP with Route 53

1. **Enable GeoIP in Route 53**:
   - Route 53 uses its own GeoIP database. No additional configuration is needed in AWS beyond setting up geolocation routing policies.

   **Output**:
   The Route 53 dashboard should show GeoIP-based routing policies active.

[Back to TOC](#table-of-contents-toc)

---

### Step 5: Create Ingress Resources in Kubernetes

1. **Create Ingress Resources** in each Kubernetes cluster (`us-west-1` and `eu-central-1`):

   **Ingress in `us-west-1` Cluster**:
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: myapp-ingress-us
     namespace: default
     annotations:
       external-dns.alpha.kubernetes.io/hostname: "us.myapp.com"
   spec:
     rules:
     - host: us.myapp.com
       http:
         paths:
         - path: /
           pathType: Prefix
           backend:
             service:
               name: myapp-service
               port:
                 number: 80
   ```

   **Ingress in `eu-central-1` Cluster**:
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: myapp-ingress-eu
     namespace: default
     annotations:
       external-dns.alpha.kubernetes.io/hostname: "eu.myapp.com"
   spec:
     rules:
     - host: eu.myapp.com
       http:
         paths:
         - path: /
           pathType: Prefix
           backend:
             service:
               name: myapp-service
               port:
                 number: 80
   ```

   **Output**:
   After deploying these resources, ExternalDNS should create corresponding DNS entries in Route 53. You can verify this by checking the Route 53 dashboard.

[Back to TOC](#table-of-contents-toc)

---

### Step 6: Test the Geolocation-Based Routing

#### Verify DNS Records

1. **Check DNS Resolution**:
   - Use the `dig` command to check the DNS resolution for `us.myapp.com` and `eu.myapp.com` from different regions:

   ```bash
   # From a North American client
   dig +short us.myapp.com
   # Output: IP of the load balancer in us-west-1

   # From a European client
   dig +short eu.myapp.com
   # Output: IP of the load balancer in eu-central-1
   ```

   **Output**:
   The IP addresses returned by the `dig` command should match the expected load balancer IPs in the respective regions.

[Back to TOC](#table-of-contents-toc)

---

#### Access the Application

1. **Access the Application**:
   - Open a browser and navigate to `us.myapp.com` and `eu.myapp.com` from different regions. Verify that traffic is correctly routed to the corresponding backend services.

   **Output**:
   You should see the application responding differently based on the user's geographical location, confirming that geolocation-based routing is working as intended.

[Back to TOC](#table-of-contents-toc)

---

### Conclusion

This step-by-step tutorial has covered the entire process of configuring ExternalDNS and GeoIP in AWS and Route 53 to enable geolocation-based routing for your Kubernetes clusters. You can now extend this setup with more complex routing rules or integrate additional services as needed.

[Back to TOC](#table-of-contents-toc)
