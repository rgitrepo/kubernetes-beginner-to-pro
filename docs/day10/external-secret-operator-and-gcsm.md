Here is the updated tutorial based on your requirements, with the project name `external-secrets-operator` and enhanced details:

---

### Tutorial: External Secret Operator (ESO) with GCP Secret Manager (GCPSM)

This tutorial will guide you through managing secrets in Kubernetes using the External Secret Operator (ESO) with **Google Cloud Secret Manager (GCPSM)**. We will cover setting up an IAM service account, storing service account credentials, configuring the Secret Store, and retrieving secrets using ExternalSecret.

---

#### Table of Contents
1. [Introduction](#introduction)
2. [Understanding the Flow](#understanding-the-flow)
3. [Prerequisites](#prerequisites)
4. [Creating an IAM Service Account](#creating-an-iam-service-account)
    - 4.1 [Assigning Roles and Permissions (GCP)](#assigning-roles-and-permissions-gcp)
    - 4.2 [Creating and Downloading the JSON Key](#creating-and-downloading-the-json-key)
5. [Complete Setup of External Secrets in Kubernetes](#complete-setup-of-external-secrets-in-kubernetes)
    - 5.1 [Storing GCP Service Account Key as a Kubernetes Secret](#storing-gcp-service-account-key-as-a-kubernetes-secret)
    - 5.2 [Configuring the Secret Store](#configuring-the-secret-store)
    - 5.3 [Creating and Applying an External Secret Resource](#creating-and-applying-an-external-secret-resource)
6. [Diagram of Connections Across Files](#diagram-of-connections-across-files)
7. [Syncing and Rotating Secrets](#syncing-and-rotating-secrets)
8. [Security and Encryption](#security-and-encryption)
9. [Conclusion](#conclusion)

---

### Introduction

The External Secret Operator (ESO) allows Kubernetes to securely fetch secrets from **Google Cloud Secret Manager (GCPSM)**. ESO automatically syncs these secrets into Kubernetes, simplifying secret management.

---

### Understanding the Flow

The process involves the following steps:
1. **Service Account Setup**: Create a GCP service account with necessary roles. For this, you create a **secret file** or **key**. 
2. **Secret Storage in Kubernetes**: Store the service account credentials as a Kubernetes secret, which ESO uses to authenticate with GCP Secret Manager.
3. **Secret Store Configuration**: Define a Secret Store in Kubernetes that references the service account secret.
4. **External Secret Creation**: Create an ExternalSecret resource in Kubernetes to fetch secrets from GCPSM and sync them into Kubernetes. It has references to **GCP Secret Manager** and to the **Secret Store**.

---

### Prerequisites

Before starting, make sure you have:
- A working Kubernetes cluster with `kubectl` access.
- A GCP account with Secret Manager API enabled.
- Basic knowledge of Kubernetes secrets and IAM roles.

---

### Creating an IAM Service Account

In this section, we will create a service account in GCP and assign the necessary permissions to access **GCP Secret Manager (GCPSM)**.

#### Assigning Roles and Permissions (GCP)

1. **Create Service Account**:

    Run the following command to create a new service account in your GCP project.

    ```bash
    gcloud iam service-accounts create my-secret-sa \
        --description="Service account for accessing secrets" \
        --display-name="my-secret-sa"
    ```

    This will create a service account named `my-secret-sa` in the GCP project `external-secrets-operator`.

2. **Grant the Secret Accessor Role**:

    Now, assign the **Secret Manager Secret Accessor** role to the service account, which allows it to access secrets in GCPSM:

    ```bash
    gcloud projects add-iam-policy-binding external-secrets-operator \
        --member="serviceAccount:my-secret-sa@external-secrets-operator.iam.gserviceaccount.com" \
        --role="roles/secretmanager.secretAccessor"
    ```

---

#### Creating and Downloading the JSON Key

Once the service account is created and permissions are assigned, you need to generate and download the **JSON key** for the service account.

```bash
gcloud iam service-accounts keys create ~/key.json \
    --iam-account=my-secret-sa@external-secrets-operator.iam.gserviceaccount.com
```

This command will create a **JSON file** (`~/key.json`) containing the credentials for the service account. These credentials will be stored as a Kubernetes secret.

---

### Complete Setup of External Secrets in Kubernetes

Now that the service account is ready, we’ll proceed to:
1. Store the JSON key in Kubernetes.
2. Configure the Secret Store.
3. Create the ExternalSecret resource to fetch secrets from GCPSM.

---

#### Storing GCP Service Account Key as a Kubernetes Secret

The GCP service account credentials (from the `key.json` file) need to be stored in Kubernetes as a secret.

**YAML File**: `gcpsm-secret.yaml`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: gcpsm-secret    # The name of the Kubernetes secret
  labels:
    type: gcpsm         # Label to categorize this secret (for GCP Secret Manager)
type: Opaque            # Opaque means this secret holds arbitrary data
stringData:
  secret-access-credentials: |-   # Storing the service account credentials from the JSON key
    {
      "type": "service_account",
      "project_id": "external-secrets-operator",  # GCP project ID from key.json
      "private_key_id": "abcdef1234567890abcdef1234567890abcdef12",  # From key.json
      "private_key": "-----BEGIN PRIVATE KEY-----\nYOUR_PRIVATE_KEY\n-----END PRIVATE KEY-----\n",  # From key.json
      "client_email": "my-secret-sa@external-secrets-operator.iam.gserviceaccount.com",  # From key.json
      "client_id": "123456789012345678901",  # From key.json
      "auth_uri": "https://accounts.google.com/o/oauth2/auth",
      "token_uri": "https://oauth2.googleapis.com/token",
      "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
      "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/my-secret-sa%40external-secrets-operator.iam.gserviceaccount.com"
    }
```

- The **key.json** values are copied into the Kubernetes secret (`gcpsm-secret.yaml`), allowing the External Secret Operator (ESO) to use these credentials to authenticate with GCP Secret Manager.

---

#### Configuring the Secret Store

Once the service account credentials are stored in Kubernetes, you need to configure the Secret Store. The Secret Store defines how Kubernetes will connect to GCPSM using the stored credentials.

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
      projectID: external-secrets-operator   # GCP project ID
      auth:
        secretRef:
          name: gcpsm-secret   # References the Kubernetes secret (gcpsm-secret.yaml)
          key: secret-access-credentials  # The field in the secret holding the credentials
```

- **Explanation**: The **`secretRef.name`** points to the Kubernetes secret (`gcpsm-secret`), and the **`secretRef.key`** field references the key within the secret (`secret-access-credentials`), which contains the service account credentials copied from `key.json`.

---

#### Creating and Applying an External Secret Resource

Next, create an External Secret resource that specifies which secret from GCP Secret Manager should be synced into Kubernetes.

**YAML File**: `my-external-secret.yaml`

```yaml
apiVersion: external-secrets.io/v1alpha1
kind: ExternalSecret
metadata:
  name: my-external-secret   # Name of the External Secret
  namespace: external-secrets
spec:
  refreshInterval: "1h"   # Specifies how often the secret should be refreshed from GCP
  secretStoreRef:
    name: gcp-secrets-store   # Refers to the Secret Store defined earlier (gcp-secret-store.yaml)
  target:
    name: my-k8s-secret   # The name of the secret to create in Kubernetes
    creationPolicy: Owner  # ESO will own the lifecycle of the secret
  data:
    - secretKey: api-key    # The key under which this secret will be stored in Kubernetes
      remoteRef:
        key: projects/external-secrets-operator/secrets/my-secret/versions/latest  # Reference to the GCP secret to fetch
```

- **Explanation**:
  - The **`remoteRef.key`** specifies the path to the secret in **GCP Secret Manager**. This format is always `projects/external-secrets-operator/secrets/<secret-name>/versions/latest`.
  - In this example:
    - **`projects/external-secrets-operator`** refers to the GCP project.
    - **`secrets/my-secret`** is the name of the secret stored in GCP.
    - **`versions/latest`** ensures that the latest version of the secret is fetched.


In the `my-external-secret.yaml` file, the fields `target.name` and `remoteRef.key` serve two distinct purposes, even though both are

 related to secrets:


#### How They Work Together: The relationship between External Secret's remoteRef.key & target.name
1. **remoteRef.key**: This field points to the **source** of the secret in **Google Cloud Secret Manager**.
   - The secret is fetched from `projects/external-secrets-operator/secrets/my-secret/versions/latest`.
   
2. **target.name**: This field defines where the fetched secret will be stored **inside Kubernetes**.
   - Once the secret is retrieved from GCP, it is stored as a Kubernetes secret named `my-k8s-secret`.

In summary:
- `remoteRef.key` tells ESO **which external secret** to fetch from **GCP Secret Manager**.
- `target.name` defines the **name of the Kubernetes secret** that will be created with the data fetched from the external source.

### Example Scenario:
- You have a secret named `my-secret` in GCP Secret Manager, and ESO will fetch the latest version of this secret.
- ESO will then create or update a Kubernetes secret called `my-k8s-secret` in your Kubernetes namespace with the content of the secret from GCP.

So, the data comes from `my-secret` in GCP, and Kubernetes will store it under `my-k8s-secret`.

---

### **Diagram of Connections Across Files**

<div style="text-align: center;">
  <img src="../../pics/external-secret.draw.io.drawio.png" alt="External Secret">
</div>


---
### Syncing and Rotating Secrets

ESO automatically syncs secrets from the cloud provider's secret manager based on the `refreshInterval`. If a secret is rotated in GCP, ESO will update the Kubernetes secret, ensuring the cluster always has the latest version.

---

### Security and Encryption

#### Encryption at Rest in GCP
All secrets in **Google Cloud Secret Manager** (GCPSM) are encrypted by default using Google-managed encryption keys. Optionally, you can use customer-managed encryption keys (CMEK) to control the encryption of your secrets.

#### Encryption in Transit
When ESO fetches the secrets from GCPSM, they are transmitted securely using **TLS encryption** to ensure that secrets are safe during transit.

#### In Kubernetes
Once ESO retrieves the secret from GCPSM, it is stored in Kubernetes as a **Kubernetes Secret**. Kubernetes secrets are base64-encoded by default but not encrypted. However, you can configure Kubernetes to encrypt secrets at rest using Kubernetes' native encryption mechanisms.

---

### Conclusion

This tutorial has provided a complete walkthrough for setting up External Secret Operator (ESO) with **GCP Secret Manager**. The process includes creating service accounts, storing credentials as Kubernetes secrets, configuring ESO, and syncing secrets from GCP into Kubernetes clusters.
