## Interview Questions - Secrets

---

### Interview Questions and Answers

1. **What is the difference between encoding and encryption?**
   - **Answer**: Encoding transforms data into a different format using a specific scheme, like Base64, and is reversible. Encryption transforms data into a secure format that can only be decrypted using a key.

2. **How are Secrets stored in Kubernetes?**
   - **Answer**: Secrets are stored encoded using Base64, not encrypted.

3. **What are some best practices for managing Kubernetes Secrets?**
   - **Answer**: Best practices include regular rotation of Secrets, limiting access to them, and avoiding storing them in version control systems.

4. **Why does Kubernetes use encoding instead of encryption for Secrets?**
   - **Answer**: Kubernetes uses encoding to avoid the complexity and overhead that encryption would add to the system, as Kubernetes is primarily focused on container orchestration rather than security.
  
5. **How do you configure a Pod to pull images from a private registry in Kubernetes?**
   - **Answer**: To pull images from a private registry, you need to create a Docker registry Secret using the `kubectl create secret docker-registry` command. Then, reference this Secret in your Pod specification using the `imagePullSecrets` field:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: my-pod
   spec:
     containers:
     - name: my-container
       image: <your-private-image>
     imagePullSecrets:
     - name: my-registry-secret
   ```

[Back to TOC](#table-of-contents)


---
