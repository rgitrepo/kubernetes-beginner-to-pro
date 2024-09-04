## External Secret Operator & GCSM in Kubernetes


#### Table of Contents
1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Setting Up the Service Account](#setting-up-the-service-account)
4. [Creating Secrets in Kubernetes](#creating-secrets-in-kubernetes)
5. [Configuring External Secret Operator](#configuring-external-secret-operator)
6. [Using GCSM with External Secret Operator](#using-gcsm-with-external-secret-operator)
7. [Rotating Secrets](#rotating-secrets)
8. [Troubleshooting and Validation](#troubleshooting-and-validation)
9. [Conclusion](#conclusion)

---

### Introduction

The External Secret Operator allows Kubernetes to fetch secrets from external secret management systems like AWS Secrets Manager, GCP Secret Manager (GCSM), and HashiCorp Vault. This centralized management of secrets is critical for improving security and consistency across environments, especially when dealing with multiple teams or clusters.

**Important Point:** The transcript emphasizes that using sealed secrets within the cluster is not always sufficient. When secrets are outsourced or stored externally, a solution like the External Secret Operator becomes necessary to securely pull these secrets into your Kubernetes cluster.

### Prerequisites

Before starting, ensure you have:
- A Kubernetes cluster with `kubectl` access
- A GCP project with Secret Manager API enabled
- Basic knowledge of Kubernetes secrets

### Setting Up the Service Account

#### 1. Create a Service Account in GCP

Creating a service account is the first step. This account will act on behalf of your Kubernetes cluster to access secrets stored in GCSM.

```bash
gcloud iam service-accounts create my-secret-sa \
    --description="Service account for accessing secrets" \
    --display-name="my-secret-sa"
```

**Why this is important:** The service account serves as the identity that your Kubernetes cluster uses to authenticate with GCSM. Without it, your cluster won't be able to securely access the secrets it needs.

#### 2. Assign Roles to the Service Account

Next, you must grant the service account the necessary permissions to access the secrets in GCSM.

```bash
gcloud projects add-iam-policy-binding <your-project-id> \
    --member="serviceAccount:my-secret-sa@<your-project-id>.iam.gserviceaccount.com" \
    --role="roles/secretmanager.secretAccessor"
```

**Why this is important:** Assigning roles ensures that your service account has the right level of access. The `secretAccessor` role is crucial because it allows the service account to retrieve secrets from GCSM securely.

#### 3. Create and Download a JSON Key

You then create a JSON key for the service account, which will be used to authenticate the Kubernetes cluster with GCSM.

```bash
gcloud iam service-accounts keys create ~/key.json \
    --iam-account=my-secret-sa@<your-project-id>.iam.gserviceaccount.com
```

**Important Point:** The transcript stresses the need to use the JSON format for the key, as other formats like `p12` are not supported by Kubernetes for this use case.

**Why this is important:** This key is your cluster's method of proving its identity to GCSM. Without this key, your cluster can't pull the secrets, making this step critical for the setup.

### Creating Secrets in Kubernetes

#### 1. Store the Service Account Key as a Secret

Now, store the `key.json` file as a Kubernetes secret. This is necessary so that the External Secret Operator can use it to authenticate with GCSM.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: gcp-secret-key
  namespace: external-secrets
type: Opaque
data:
  gcp-key.json: <base64-encoded-json-key>
```

Apply this secret:

```bash
kubectl apply -f gcp-secret-key.yaml
```

**Why this is important:** Storing the service account key as a Kubernetes secret ensures that it remains secure and is only accessible by the External Secret Operator. This step is critical for maintaining the security of your secrets and their access.

### Configuring External Secret Operator

#### 1. Install External Secret Operator

The External Secret Operator is installed using Helm, which allows Kubernetes to fetch secrets from external sources like GCSM.

```bash
helm repo add external-secrets https://charts.external-secrets.io
helm repo update
helm install external-secrets external-secrets/external-secrets
```

**Why this is important:** Installing the External Secret Operator is a necessary step to allow your Kubernetes cluster to interact with external secret management services like GCSM. Without this operator, your cluster wouldn't be able to pull secrets from GCSM.

#### 2. Configure the Secret Store

Now, configure the `SecretStore` which tells the External Secret Operator where and how to fetch secrets from GCSM.

```yaml
apiVersion: external-secrets.io/v1alpha1
kind: SecretStore
metadata:
  name: gcp-secrets-store
  namespace: external-secrets
spec:
  provider:
    gcp:
      projectID: <your-project-id>
      auth:
        secretRef:
          name: gcp-secret-key
          key: gcp-key.json
```

Apply the SecretStore:

```bash
kubectl apply -f gcp-secrets-store.yaml
```

**Why this is important:** The `SecretStore` defines the connection between your Kubernetes cluster and the GCSM. It ensures that the External Secret Operator knows how to authenticate and which secrets to pull.

**Important Point:** The transcript highlights the need for careful configuration of this step, as it's crucial for ensuring the security and proper functioning of the secret-fetching process.

### Using GCSM with External Secret Operator

#### 1. Create an External Secret Resource

This resource tells Kubernetes to fetch the secret from GCSM and store it as a Kubernetes secret.

```yaml
apiVersion: external-secrets.io/v1alpha1
kind: ExternalSecret
metadata:
  name: my-external-secret
  namespace: external-secrets
spec:
  refreshInterval: "1h"
  secretStoreRef:
    name: gcp-secrets-store
  target:
    name: my-k8s-secret
    creationPolicy: Owner
  data:
    - secretKey: api-key
      remoteRef:
        key: projects/<your-project-id>/secrets/my-secret/versions/latest
```

Apply the ExternalSecret:

```bash
kubectl apply -f my-external-secret.yaml
```

**Why this is important:** Creating an External Secret resource is the core of this tutorial. It directly connects your Kubernetes cluster with GCSM, allowing secrets to be securely fetched and used within your cluster. The `refreshInterval` setting is important as it defines how often the secret should be synced, which is critical for keeping your secrets up-to-date.

### Rotating Secrets

You can rotate secrets by updating them in GCSM. The External Secret Operator will automatically pull the updated secret based on the refresh interval defined.

**Important Point:** The transcript emphasizes the importance of secret rotation policies, especially in companies that require secrets to be rotated regularly, such as every 30 days.

**Why this is important:** Secret rotation is vital for maintaining security. Regularly rotating secrets reduces the risk of unauthorized access if a secret is compromised.

### Troubleshooting and Validation

#### 1. Check for Sync Errors

If the secret isn't created, check for sync errors:

```bash
kubectl describe externalsecret my-external-secret -n external-secrets
```

**Why this is important:** Troubleshooting sync errors is crucial for ensuring that your secrets are correctly pulled and stored in Kubernetes. Issues here might indicate problems with authentication or configuration.

#### 2. Ensure Valid Credentials

If there are issues with authentication, verify that the service account has the correct roles and that the JSON key is correctly encoded in the Kubernetes secret.

**Important Point:** According to the transcript, common issues like sync errors are often due to incorrect credentials or improperly configured JSON keys.

### Conclusion

The External Secret Operator provides a secure and automated way to manage secrets across different environments by fetching them from a centralized store like GCSM. By following this detailed tutorial, you should have a fully functioning setup that securely manages and rotates secrets in your Kubernetes cluster.
