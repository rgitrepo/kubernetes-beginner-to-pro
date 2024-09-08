
## Tutorial: External Secret Operator (ESO) with AWS, GCP, and Azure

This tutorial provides a step-by-step guide for using the External Secret Operator (ESO) with AWS, GCP, and Azure, focusing on managing secrets securely in Kubernetes. The following explanations include detailed YAML files with comments to enhance your understanding of every step.

#### Table of Contents
1. [Introduction](#introduction)
2. [Understanding the Flow](#understanding-the-flow)
3. [Prerequisites](#prerequisites)
4. [Creating an IAM Service Account](#creating-an-iam-service-account)
    - 4.1 [Assigning Roles and Permissions](#41-assigning-roles-and-permissions)
    - 4.2 [Creating and Downloading the JSON Key](#42-creating-and-downloading-the-json-key)
5. [Storing Service Account Key as a Kubernetes Secret](#storing-service-account-key-as-a-kubernetes-secret)
6. [Configuring the Secret Store](#configuring-the-secret-store)
7. [Setting Up External Secrets](#setting-up-external-secrets)
8. [Syncing and Rotating Secrets](#syncing-and-rotating-secrets)
9. [Security and Encryption](#security-and-encryption)
10. [Use Cases and Applications](#use-cases-and-applications)
11. [Troubleshooting Common Issues](#troubleshooting-common-issues)
12. [Conclusion](#conclusion)

---

### Introduction

The External Secret Operator (ESO) allows Kubernetes to fetch secrets from cloud secret management systems like AWS Secrets Manager, Google Cloud Secret Manager (GCSM), and Azure Key Vault. By doing so, you can centralize your secret management while ensuring secrets are automatically synced into your Kubernetes cluster.

[Back to Table of Contents](#table-of-contents)

---

### Understanding the Flow

**Overview of the Flow**:
1. **IAM Service Account Creation**: In GCP, AWS, or Azure, a service account is created with the necessary permissions to access cloud-based secrets.
2. **Secret Store**: ESO uses a Kubernetes `SecretStore` resource to manage how Kubernetes will connect to cloud providers to fetch secrets.
3. **External Secret**: A Kubernetes `ExternalSecret` resource defines which cloud-based secret is pulled into Kubernetes as a native secret.

Each step involves critical security concepts and ensures that Kubernetes can securely retrieve the secrets.

[Back to Table of Contents](#table-of-contents)

---

### Prerequisites

Before proceeding, ensure you have:
- A working Kubernetes cluster with access to `kubectl`.
- AWS, GCP, or Azure account with Secret Manager API enabled.
- A basic understanding of Kubernetes secrets.

[Back to Table of Contents](#table-of-contents)

---

### Creating an IAM Service Account

#### 4.1 Assigning Roles and Permissions

This step covers how to create a service account with the appropriate roles and permissions to access cloud secrets.

**For GCP**:
1. **Create the Service Account**:

    ```bash
    gcloud iam service-accounts create my-secret-sa \
        --description="Service account for accessing secrets" \
        --display-name="my-secret-sa"
    ```
    > **Explanation**: This creates a service account that Kubernetes will use to securely authenticate with Google Cloud Secret Manager (GCSM).

2. **Grant Permissions**:

    ```bash
    gcloud projects add-iam-policy-binding <your-project-id> \
        --member="serviceAccount:my-secret-sa@<your-project-id>.iam.gserviceaccount.com" \
        --role="roles/secretmanager.secretAccessor"
    ```
    > **Explanation**: The `secretAccessor` role is granted to the service account, allowing it to retrieve secrets from GCSM.

**For AWS**:
1. **Create an IAM Policy**:

    ```json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": "secretsmanager:GetSecretValue",
          "Resource": "*"
        }
      ]
    }
    ```
    > **Explanation**: This policy allows the IAM role to access AWS Secrets Manager to retrieve secrets.

2. **Attach the Policy**: Attach the IAM policy to the Kubernetes worker node's IAM role.

[Back to Table of Contents](#table-of-contents)

---

#### 4.2 Creating and Downloading the JSON Key

After assigning permissions, create a JSON key for the service account.

**For GCP**:
```bash
gcloud iam service-accounts keys create ~/key.json \
    --iam-account=my-secret-sa@<your-project-id>.iam.gserviceaccount.com
```
> **Explanation**: This key will be stored as a Kubernetes secret and used to authenticate the cluster with GCSM. The file is crucial for secure authentication.

[Back to Table of Contents](#table-of-contents)

---

### Storing Service Account Key as a Kubernetes Secret

Now that you have the service account key, you’ll store it securely in Kubernetes.

**YAML File**: `gcp-secret-key.yaml`
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: gcp-secret-key    # This is the name of the secret in Kubernetes
  namespace: external-secrets   # Ensure this namespace is created
type: Opaque
data:
  gcp-key.json: <base64-encoded-json-key>  # The key.json file, base64 encoded
```
**Command**:
```bash
kubectl apply -f gcp-secret-key.yaml
```
> **Explanation**: Here, we store the service account key in a Kubernetes secret. It is stored in base64 encoding for security reasons.

[Back to Table of Contents](#table-of-contents)

---

### Configuring the Secret Store

The Secret Store allows Kubernetes to connect to GCP, AWS, or Azure securely and retrieve secrets.

**YAML File**: `gcp-secret-store.yaml`
```yaml
apiVersion: external-secrets.io/v1alpha1
kind: SecretStore
metadata:
  name: gcp-secrets-store  # Name of the Secret Store
  namespace: external-secrets  # Namespace where this resource exists
spec:
  provider:
    gcp:
      projectID: <your-project-id>  # GCP project ID where the secrets are stored
      auth:
        secretRef:
          name: gcp-secret-key  # The secret in Kubernetes containing the GCP service account key
          key: gcp-key.json  # Refers to the actual key in the secret
```

**Command**:
```bash
kubectl apply -f gcp-secret-store.yaml
```

> **Explanation**: This Secret Store configuration tells the External Secret Operator how to authenticate with GCSM using the service account key stored in Kubernetes. It's a necessary step for allowing Kubernetes to securely pull secrets from the external cloud provider.

[Back to Table of Contents](#table-of-contents)

---

### Setting Up External Secrets

The `ExternalSecret` resource is used to sync a secret from GCSM into Kubernetes.

**YAML File**: `my-external-secret.yaml`
```yaml
apiVersion: external-secrets.io/v1alpha1
kind: ExternalSecret
metadata:
  name: my-external-secret  # Name of the external secret in Kubernetes
  namespace: external-secrets  # Namespace where the secret will reside
spec:
  refreshInterval: "1h"  # How often the secret should be refreshed
  secretStoreRef:
    name: gcp-secrets-store  # Refers to the Secret Store we created earlier
  target:
    name: my-k8s-secret  # This is the name of the secret that will be created in Kubernetes
    creationPolicy: Owner  # Ensures Kubernetes owns and manages the secret
  data:
    - secretKey: api-key  # The key under which this secret will be stored in Kubernetes
      remoteRef:
        key: projects/<your-project-id>/secrets/my-secret/versions/latest  # Reference to the GCSM secret
```

**Command**:
```bash
kubectl apply -f my-external-secret.yaml
```

> **Explanation**: This YAML file defines an external secret in Kubernetes. It points to the cloud-based secret (in this case, GCSM) and tells Kubernetes to sync this secret into its environment. The `refreshInterval` ensures the secret is updated regularly, while the `target` specifies where the secret will reside in Kubernetes.

[Back to Table of Contents](#table-of-contents)


---

### GCP Service Account Authentication with ESO

This example demonstrates how to authenticate with Google Cloud Secret Manager (GCSM) using a GCP Service Account. The Service Account credentials are stored in Kubernetes as a secret, which ESO will use to fetch secrets from GCSM.

**GCP Service Account Authentication Overview**:
- **Service Account**: A long-lived credential (static) JSON file.
- **Kind=Secret**: This Kubernetes secret contains the GCP service account JSON, allowing ESO to authenticate with GCSM and retrieve secrets.
- **Secret Manager Accessor Role**: This role is required for the service account to read secrets from GCSM.

---

### YAML File Example: Storing the GCP Service Account in Kubernetes

**YAML File**: `gcpsm-secret.yaml`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: gcpsm-secret    # Name of the Kubernetes secret
  labels:
    type: gcpsm         # A label to easily identify this as GCP Secret Manager-related
type: Opaque            # The type of secret (Opaque indicates arbitrary user-defined data)
stringData:
  secret-access-credentials: |-   # The GCP service account credentials are stored as a key-value pair
    {
      "type": "service_account",    # Defines the type of the GCP account as a service account
      "project_id": "external-secrets-operator",  # GCP project where the service account exists
      "private_key_id": "",   # Private key ID, will be auto-generated when the key is created
      "private_key": "-----BEGIN PRIVATE KEY-----\nA key\n-----END PRIVATE KEY-----\n",  # The private key for secure access
      "client_email": "test-service-account@external-secrets-operator.iam.gserviceaccount.com",  # Service account email
      "client_id": "client ID",   # Unique client ID for the service account
      "auth_uri": "https://accounts.google.com/o/oauth2/auth",  # OAuth 2.0 authorization URI
      "token_uri": "https://oauth2.googleapis.com/token",  # URI to obtain the access token
      "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",  # URI for the provider's cert
      "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/test-service-account%40external-secrets-operator.iam.gserviceaccount.com"  # Service account's certificate URI
    }
```

### Explanation and Connections to Other YAML Files

1. **`apiVersion: v1` and `kind: Secret`**: This defines the Kubernetes object as a secret. In other parts of the tutorial (e.g., the [Storing Service Account Key](#storing-service-account-key-as-a-kubernetes-secret) section), this YAML file represents the secret that will hold the GCP Service Account credentials. ESO will later use this to authenticate with GCSM.

2. **`metadata.name: gcpsm-secret`**: This is the name of the secret. In your Secret Store YAML (e.g., [Configuring the Secret Store](#configuring-the-secret-store) section), you will reference this secret under `spec.provider.gcp.auth.secretRef.name`, indicating that this secret contains the necessary credentials for GCP.

3. **`labels`**: Labels provide a way to easily identify this secret as part of GCP Secret Manager. While not critical, it's good practice to use labels to categorize resources. In larger clusters, labels can help distinguish different types of resources.

4. **`type: Opaque`**: The `Opaque` type indicates that the secret holds arbitrary data, like credentials or keys. This is consistent with how Kubernetes manages secrets for various services. It corresponds to the secret type used in previous examples (e.g., [Storing Service Account Key](#storing-service-account-key-as-a-kubernetes-secret)).

5. **`stringData.secret-access-credentials`**: The key `secret-access-credentials` holds the entire GCP service account JSON. In the Secret Store YAML file ([Configuring the Secret Store](#configuring-the-secret-store)), you’ll notice this field is accessed through `spec.provider.gcp.auth.secretRef.key` to authenticate with GCP.

6. **Connection to Secret Store**: In the Secret Store YAML, this secret (`gcpsm-secret`) is referenced in this manner:
    ```yaml
    auth:
      secretRef:
        name: gcpsm-secret   # Refers to this secret created here
        key: secret-access-credentials  # Refers to the key in the secret containing the JSON
    ```

7. **GCP Service Account JSON**:
   - **`type`**: Specifies that this is a GCP service account.
   - **`project_id`**: Links to your GCP project where the service account is created. The `project_id` is critical because it determines the context in which the service account operates. This must match the project where your secrets are stored.
   - **`private_key` and `client_email`**: The `private_key` allows secure authentication with GCP, and `client_email` defines the service account's identity. ESO will use these credentials to authenticate.
   - **OAuth and Token URIs**: These URIs handle authentication and token exchange for secure communication with GCP.

---

### Next Steps: Connecting the Secret to External Secrets

Once the service account is stored in Kubernetes as a secret, the next step is to configure the `SecretStore` YAML and link it to this secret to authenticate with GCSM.

Refer back to the [Configuring the Secret Store](#configuring-the-secret-store) section, where you would link the secret in the following way:

**YAML Snippet**:
```yaml
auth:
  secretRef:
    name: gcpsm-secret   # Refers to the Kubernetes secret created with the GCP credentials
    key: secret-access-credentials  # Refers to the JSON key within the secret
```

By doing this, you ensure that the External Secret Operator can use the `gcpsm-secret` to authenticate with GCP and fetch secrets securely.

[Back to Table of Contents](#table-of-contents)

---



### Syncing and Rotating Secrets

ESO allows for automatic secret rotation. When secrets in GCSM or other cloud services are rotated, the External Secret Operator will sync the updated secret into Kubernetes based on the `refreshInterval`.

> **Important**: Ensure that your service account permissions allow secret retrieval even after secret rotation.

[Back to Table of Contents](#table-of-contents)

---

### Security and Encryption

Secrets stored in cloud secret managers (GCSM, AWS Secrets Manager, Azure Key Vault) are encrypted at rest by the cloud provider. When these secrets are synced to Kubernetes, they are base64 encoded, but not encrypted unless explicitly configured.

- **Encryption at Rest**: Managed by the cloud provider.
- **In-Cluster Security**: Secrets are base64 encoded. If further encryption is needed, consider using tools like Sealed Secrets.

[Back to Table of Contents](#table-of-contents)

---

### Troubleshooting Common Issues

1. **Invalid Credentials**:
   - Ensure that the service account key has the proper roles and permissions

.
   - Verify that the JSON key is correctly stored in Kubernetes as a secret.

2. **Sync Errors**:
   - Use the following command to check for errors:

     ```bash
     kubectl describe externalsecret my-external-secret -n external-secrets
     ```

3. **Secret Rotation Issues**:
   - Check that your cloud provider's secret manager is configured to rotate secrets correctly.
   - Ensure the `refreshInterval` in the `ExternalSecret` YAML file is set appropriately.

[Back to Table of Contents](#table-of-contents)

---

### Conclusion

By following the steps in this tutorial, you will have a fully functioning setup for securely managing and rotating secrets using the External Secret Operator with GCP, AWS, and Azure. The examples and detailed YAML files provided should give you a strong foundation in understanding how to set up and troubleshoot this powerful Kubernetes feature.

[Back to Table of Contents](#table-of-contents)

