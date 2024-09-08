
### Tutorial: External Secret Operator (ESO) with AWS, GCP, and Azure

This tutorial covers the process of managing secrets in Kubernetes using the External Secret Operator (ESO) with cloud secret management systems like AWS Secrets Manager, Google Cloud Secret Manager (GCSM), and Azure Key Vault. Follow the steps to create service accounts, store secrets, configure ESO, and securely access external secrets from these cloud providers.

#### Table of Contents
1. [Introduction](#introduction)
2. [Understanding the Flow](#understanding-the-flow)
3. [Prerequisites](#prerequisites)
4. [Creating an IAM Service Account](#creating-an-iam-service-account)
    - 4.1 [Assigning Roles and Permissions (GCP, AWS, Azure)](#assigning-roles-and-permissions-gcp-aws-azure)
    - 4.2 [Creating and Downloading the JSON Key](#creating-and-downloading-the-json-key)
5. [Complete Setup of External Secrets in Kubernetes](#complete-setup-of-external-secrets-in-kubernetes)
    - 5.1 [Storing GCP Service Account Key as a Kubernetes Secret](#storing-gcp-service-account-key-as-a-kubernetes-secret)
    - 5.2 [Configuring the Secret Store](#configuring-the-secret-store)
    - 5.3 [Creating and Applying an External Secret Resource](#creating-and-applying-an-external-secret-resource)
6. [Syncing and Rotating Secrets](#syncing-and-rotating-secrets)
7. [Security and Encryption](#security-and-encryption)
8. [Conclusion](#conclusion)

---

### Introduction

The External Secret Operator (ESO) allows Kubernetes to fetch secrets from cloud secret management systems like AWS Secrets Manager, GCP Secret Manager (GCSM), and Azure Key Vault. ESO automatically syncs these secrets into Kubernetes, helping teams manage and rotate secrets securely across multiple environments.

[Back to Table of Contents](#table-of-contents)

---

### Understanding the Flow

To understand how ESO works, let’s break down the flow:
1. **Service Account Setup**: Create a service account with the necessary roles in GCP, AWS, or Azure to access their respective secret management systems.
2. **Secret Storage in Kubernetes**: Store the service account credentials as a Kubernetes secret, which ESO uses to authenticate with the cloud provider.
3. **Secret Store Configuration**: Define a Secret Store in Kubernetes that points to the cloud provider's secret manager and links to the service account secret.
4. **External Secret Creation**: Create an ExternalSecret resource in Kubernetes to fetch secrets from the cloud provider and sync them into Kubernetes as native secrets.

[Back to Table of Contents](#table-of-contents)

---

### Prerequisites

Before you begin, make sure you have:
- A Kubernetes cluster with `kubectl` access.
- An account in AWS, GCP, or Azure with Secret Manager API enabled.
- Basic knowledge of Kubernetes secrets and cloud service identities (IAM roles, service accounts).

[Back to Table of Contents](#table-of-contents)

---

### Creating an IAM Service Account

This section explains how to create an IAM service account for GCP, AWS, and Azure. The service account will allow Kubernetes to authenticate with the respective cloud provider's secret manager.

#### Assigning Roles and Permissions (GCP, AWS, Azure)

**For GCP**:
1. **Create Service Account**:

    ```bash
    gcloud iam service-accounts create my-secret-sa \
        --description="Service account for accessing secrets" \
        --display-name="my-secret-sa"
    ```

2. **Grant the Secret Accessor Role**:

    ```bash
    gcloud projects add-iam-policy-binding <your-project-id> \
        --member="serviceAccount:my-secret-sa@<your-project-id>.iam.gserviceaccount.com" \
        --role="roles/secretmanager.secretAccessor"
    ```

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

2. **Attach the Policy**: Attach this IAM policy to the Kubernetes worker node’s IAM role to allow access to AWS Secrets Manager.

**For Azure**:
1. **Create Managed Identity and Assign Roles**:

    ```bash
    az ad sp create-for-rbac --name my-key-vault-reader \
        --role "Key Vault Reader" --scopes /subscriptions/<subscription-id>
    ```

> **Explanation**: Each service provider requires specific permissions to allow Kubernetes to retrieve secrets. You need to ensure the service account has the appropriate roles to access the secret manager.

[Back to Table of Contents](#table-of-contents)

---

#### Creating and Downloading the JSON Key

Once the service account is created and permissions are assigned, generate and download the service account key.

**For GCP**:

```bash
gcloud iam service-accounts keys create ~/key.json \
    --iam-account=my-secret-sa@<your-project-id>.iam.gserviceaccount.com
```

> **Explanation**: This command generates a JSON file containing the credentials for the service account, which will be used by Kubernetes to authenticate with GCP Secret Manager. This file is critical and will be stored as a Kubernetes secret.

[Back to Table of Contents](#table-of-contents)

---

### Complete Setup of External Secrets in Kubernetes

This section will cover the complete setup process, including storing the service account key, configuring the Secret Store, and creating an External Secret resource.

#### Storing GCP Service Account Key as a Kubernetes Secret

The JSON key generated earlier needs to be stored securely in Kubernetes as a secret, which will be used by the External Secret Operator (ESO) to authenticate with GCP.

**YAML File**: `gcpsm-secret.yaml`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: gcpsm-secret    # The name of the Kubernetes secret
  labels:
    type: gcpsm         # Label indicating this is for GCP Secret Manager
type: Opaque            # Opaque means this secret holds arbitrary data
stringData:
  secret-access-credentials: |-   # The actual service account credentials from the JSON key
    {
      "type": "service_account",
      "project_id": "external-secrets-operator",  # Your GCP project ID
      "private_key_id": "private-key-id",   # Private key ID from the JSON file
      "private_key": "-----BEGIN PRIVATE KEY-----\nYOUR_PRIVATE_KEY\n-----END PRIVATE KEY-----\n",  # The private key
      "client_email": "your-service-account@external-secrets-operator.iam.gserviceaccount.com",  # The service account email
      "client_id": "client-id",   # Client ID from the JSON file
      "auth_uri": "https://accounts.google.com/o/oauth2/auth",
      "token_uri": "https://oauth2.googleapis.com/token",
      "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
      "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/your-service-account%40external-secrets-operator.iam.gserviceaccount.com"
    }
```

**Command**:
```bash
kubectl apply -f gcpsm-secret.yaml
```

> **Explanation**: This Kubernetes secret holds the GCP service account credentials, which will be referenced later by the Secret Store to authenticate Kubernetes with GCP Secret Manager.

[Back to Table of Contents](#table-of-contents)

---

#### Configuring the Secret Store

Once the service account credentials are stored in Kubernetes, you need to configure a Secret Store that tells ESO how to authenticate with GCP and retrieve secrets.

**YAML File**: `gcp-secret-store.yaml`

```yaml
apiVersion: external-secrets.io/v1alpha1
kind: SecretStore
metadata:
  name: gcp-secrets-store   # The name of the Secret Store
  namespace: external-secrets  # The namespace where ESO is installed
spec:
  provider:
    gcp:
      projectID: external-secrets-operator   # GCP project where the secrets are stored
      auth:
        secretRef:
          name: gcpsm-secret   # Refers to the Kubernetes secret holding the service account credentials
          key: secret-access-credentials  # Refers to the key within the secret containing the JSON credentials
```

**Command**:
```bash
kubectl apply -f gcp-secret-store.yaml
```

> **Explanation**: The Secret Store defines how Kubernetes will connect to GCP Secret Manager using the service account credentials stored in the Kubernetes secret (`gcpsm-secret`). ESO will use this configuration to retrieve secrets from GCP.

[Back to Table of Contents](#table-of-contents)

---

#### Creating and Applying an External Secret Resource

Now, create an External Secret resource in Kubernetes. This resource specifies which secret from GCP Secret Manager should be synced into Kubernetes.

**YAML File**: `my-external-secret.yaml`

```yaml
apiVersion: external-secrets.io/v1alpha1
kind: ExternalSecret
metadata:
  name: my-external-secret   # The name of the External Secret
  namespace: external-secrets
spec

:
  refreshInterval: "1h"   # Specifies how often the secret should be refreshed from GCP
  secretStoreRef:
    name: gcp-secrets-store   # Refers to the Secret Store defined earlier
  target:
    name: my-k8s-secret   # The name of the secret to create in Kubernetes
    creationPolicy: Owner
  data:
    - secretKey: api-key    # Key under which this secret will be stored in Kubernetes
      remoteRef:
        key: projects/<your-project-id>/secrets/my-secret/versions/latest  # Reference to the GCP secret to fetch
```

**Command**:
```bash
kubectl apply -f my-external-secret.yaml
```

> **Explanation**: This External Secret resource pulls the secret from GCP Secret Manager and stores it in Kubernetes as `my-k8s-secret`. The `refreshInterval` ensures that the secret is updated every hour.

[Back to Table of Contents](#table-of-contents)

---

### Syncing and Rotating Secrets

ESO automatically syncs secrets from the cloud provider's secret manager based on the `refreshInterval` specified in the External Secret resource. If a secret is rotated in GCP, AWS, or Azure, ESO will update the Kubernetes secret accordingly, ensuring that your cluster always has the most up-to-date version.

[Back to Table of Contents](#table-of-contents)

---

### Security and Encryption

Secrets stored in GCP, AWS, or Azure are encrypted at rest by the respective cloud providers. When these secrets are pulled into Kubernetes, they are base64 encoded, but not encrypted. For additional security, consider using Kubernetes tools like Sealed Secrets or additional encryption methods.

[Back to Table of Contents](#table-of-contents)

---

### Conclusion

This tutorial has provided a complete walkthrough for setting up External Secret Operator (ESO) with GCP, AWS, and Azure. From creating service accounts to configuring Kubernetes secrets and syncing them with cloud providers, you now have a comprehensive guide to securely manage secrets in your Kubernetes clusters.

[Back to Table of Contents](#table-of-contents)

