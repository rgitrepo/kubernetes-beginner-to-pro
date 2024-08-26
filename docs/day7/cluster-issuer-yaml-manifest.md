Here’s an example YAML manifest file for a `ClusterIssuer` in Kubernetes, configured to use Let's Encrypt to issue certificates:

### ClusterIssuer YAML Manifest

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
  namespace: cert-manager
spec:
  acme:
    # The ACME server URL. For Let's Encrypt production, use:
    server: https://acme-v02.api.letsencrypt.org/directory
    # Your email address to receive notifications from Let's Encrypt
    email: your-email@example.com
    # Name of the secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-prod
    # Enable the HTTP-01 challenge provider
    solvers:
    - http01:
        ingress:
          class: nginx
```

### Key Sections of the Manifest:

- **`apiVersion`**: Specifies the API version for Cert-Manager resources. For Cert-Manager v1.13.0 and above, this should be `cert-manager.io/v1`.

- **`kind`**: Defines the type of Kubernetes resource. Here, it's a `ClusterIssuer`, which issues certificates for the entire cluster.

- **`metadata`**: Contains metadata about the resource, including the name of the `ClusterIssuer` and its namespace. While `ClusterIssuer` is a cluster-wide resource and does not technically require a namespace, including one for consistency is common practice.

- **`spec.acme.server`**: The ACME server's URL. For production, use Let's Encrypt's live server URL. For staging/testing, you can use the staging server at `https://acme-staging-v02.api.letsencrypt.org/directory`.

- **`spec.acme.email`**: Your email address where Let's Encrypt will send important notifications (like when your certificates are about to expire).

- **`spec.acme.privateKeySecretRef.name`**: The name of the Kubernetes secret where the private key for your ACME account will be stored.

- **`spec.acme.solvers`**: Defines how the challenges for certificate validation are solved. In this example, the HTTP-01 challenge is used with an ingress class named `nginx`.

### How to Apply This Manifest

1. Save the above YAML content to a file, e.g., `cluster-issuer.yaml`.
2. Apply it to your Kubernetes cluster using the `kubectl` command:

   ```bash
   kubectl apply -f cluster-issuer.yaml
   ```

This will create a `ClusterIssuer` named `letsencrypt-prod` in your Kubernetes cluster, which you can then use to issue certificates across your cluster.

If you need a staging environment for testing purposes, replace the `server` URL with Let's Encrypt's staging URL and consider renaming the `ClusterIssuer` to something like `letsencrypt-staging`.
