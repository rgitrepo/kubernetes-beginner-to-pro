
# Tutorial: Adding a Certificate and Integrating it into Ingress Using Cert-Manager

This tutorial will guide you through the process of adding a certificate in a Kubernetes cluster using Cert-Manager and later integrating that certificate into an Ingress resource. The steps are based on the explanation from the provided transcript.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Step 1: Install Cert-Manager](#step-1-install-cert-manager)
3. [Step 2: Create a ClusterIssuer](#step-2-create-a-clusterissuer)
4. [Step 3: Verify the ClusterIssuer](#step-3-verify-the-clusterissuer)
5. [Step 4: Request a Certificate](#step-4-request-a-certificate)
6. [Step 5: Verify the Certificate](#step-5-verify-the-certificate)
7. [Step 6: Integrate the Certificate into Ingress](#step-6-integrate-the-certificate-into-ingress)
8. [Step 7: Test the Setup](#step-7-test-the-setup)
9. [Troubleshooting](#troubleshooting)
10. [Conclusion](#conclusion)

---

## Prerequisites
- A Kubernetes cluster with `kubectl` configured.
- Cert-Manager installed in your cluster.
- An Ingress controller (e.g., NGINX) set up in your cluster.

[Back to TOC](#table-of-contents)

---

## Step 1: Install Cert-Manager

If you haven’t installed Cert-Manager, you can do so by following these steps:

```bash
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.13.0/cert-manager.yaml
```

Verify the installation:

```bash
kubectl get pods --namespace cert-manager
```

[Back to TOC](#table-of-contents)

---

## Step 2: Create a ClusterIssuer

A `ClusterIssuer` is a resource in Kubernetes that represents a certificate authority that Cert-Manager can use to issue certificates. We will use Let's Encrypt as the certificate authority in this example.

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your-email@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
```

Apply this configuration:

```bash
kubectl apply -f cluster-issuer.yaml
```

[Back to TOC](#table-of-contents)

---

## Step 3: Verify the ClusterIssuer

Before proceeding with certificate requests, it's important to verify that the `ClusterIssuer` has been correctly created and is ready to use. You can do this by running the following command:

```bash
kubectl get clusterissuer letsencrypt-prod -o yaml
```

#### What This Command Does:
- **`kubectl get clusterissuer`**: This command retrieves information about the `ClusterIssuer` resources in your cluster. It’s a useful way to confirm that your `ClusterIssuer` was successfully created.
- **`letsencrypt-prod`**: This is the name of the specific `ClusterIssuer` you want to inspect. Replace it with the appropriate name if you used a different name.
- **`-o yaml`**: This flag outputs the detailed YAML configuration of the `ClusterIssuer`, showing you all its attributes, including the status. This helps verify that the `ClusterIssuer` is correctly configured and that there are no issues before proceeding to request certificates.

[Back to TOC](#table-of-contents)

---

## Step 4: Request a Certificate

Now that the `ClusterIssuer` is set up and verified, we can request a certificate. We’ll create a `Certificate` resource that references the `ClusterIssuer` and specifies the domain names for which we want the certificate.

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: example-com-tls
  namespace: default
spec:
  secretName: example-com-tls
  dnsNames:
  - example.com
  - www.example.com
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
```

Apply this configuration:

```bash
kubectl apply -f certificate.yaml
```

This configuration will:
- Request a certificate for `example.com` and `www.example.com`.
- Store the certificate and private key in a secret named `example-com-tls` in the `default` namespace.

[Back to TOC](#table-of-contents)

---

## Step 5: Verify the Certificate

After a few moments, you can verify that the certificate has been issued and stored in the secret:

```bash
kubectl get secret example-com-tls -o yaml
```

To further inspect the certificate, you can use the `kubectl describe` command:

```bash
kubectl describe secret example-com-tls
```

#### Example Output:
```plaintext
Name:         example-com-tls
Namespace:    default
Labels:       <none>
Annotations:  cert-manager.io/alt-names: example.com,www.example.com
              cert-manager.io/certificate-name: example-com-tls
              cert-manager.io/common-name: example.com
              cert-manager.io/ip-sans: 
              cert-manager.io/issuer-group: cert-manager.io
              cert-manager.io/issuer-kind: ClusterIssuer
              cert-manager.io/issuer-name: letsencrypt-prod
              cert-manager.io/uri-sans: 

Type:  kubernetes.io/tls

Data
====
tls.crt:  3647 bytes
tls.key:  1675 bytes
```

### Edit the Secret to View Encoded Data
To view the actual encoded certificate and key data, you can use the `kubectl edit` command:

```bash
kubectl edit secret example-com-tls
```

This command will open the secret in your default editor, where you can see the encoded data under `tls.crt` and `tls.key`. Here's what you might see:

```yaml
apiVersion: v1
data:
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCg==
  tls.key: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQo==
kind: Secret
metadata:
  name: example-com-tls
  namespace: default
type: kubernetes.io/tls
```

The `tls.crt` and `tls.key` values are base64 encoded, representing the certificate and private key, respectively.

[Back to TOC](#table-of-contents)

---

## Step 6: Integrate the Certificate into Ingress

To use the certificate with your Ingress resource, you need to modify your Ingress configuration to reference the certificate secret.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  namespace: default
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - example.com
    - www.example.com
    secretName: example-com-tls
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: example-service
            port:
              number: 80
  - host: www.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: example-service
            port:
              number: 80
```

Apply this configuration:

```bash
kubectl apply -f ingress.yaml
```

### Explanation:
- **Annotations**: The annotation `cert-manager.io/cluster-issuer: "letsencrypt-prod"` tells Cert-Manager to use the specified `ClusterIssuer` to manage the certificates for this Ingress.
- **TLS Block**: The `tls` block specifies the domains (`hosts`) and the secret (`secretName`) where the certificate is stored. Adding this TLS blocck of tls, hosts, secretname adds a certificate.
- **Rules**: The rules define the routing for your domain, directing traffic to the appropriate backend service.

[Back to TOC](#table-of-contents)

---

## Step 7: Test the Setup

Once the Ingress is configured and applied, your domain (e.g., `example.com`) should be accessible over HTTPS using the certificate managed by Cert-Manager.

[Back to TOC](#table-of-contents)

---

## Troubleshooting

- If you encounter any issues, check the logs of the Cert-Manager controller to identify what might be going wrong:
  
  ```bash
  kubectl logs -n cert-manager -l app=cert-manager
  ```

- Ensure that your DNS records point to the correct Ingress controller IP address.

[Back to TOC](#table-of-contents)

---

## Conclusion

This tutorial demonstrated how to add a certificate to a Kubernetes cluster using Cert-Manager, verify it using the `kubectl get clusterissuer` command, inspect it using `kubectl describe secret`, and view the encoded data using `kubectl edit secret`. You also learned how to integrate that certificate into an Ingress resource. Cert-Manager automates the management of SSL/TLS certificates, ensuring that your applications remain secure and compliant with minimal effort.

[Back to TOC](#table-of-contents)
