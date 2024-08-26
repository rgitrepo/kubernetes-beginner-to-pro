## Public & Private Keys

### Table of Contents (TOC)

1. [Introduction to Public and Private Keys](#introduction-to-public-and-private-keys)
2. [How Public and Private Keys Work](#how-public-and-private-keys-work)
3. [Public Key Infrastructure (PKI) and Certificate Authorities (CA)](#public-key-infrastructure-pki-and-certificate-authorities-ca)
4. [Communication Process Using Public and Private Keys](#communication-process-using-public-and-private-keys)

---

### 1. [Introduction to Public and Private Keys](#introduction-to-public-and-private-keys)

Public and private keys are fundamental to secure communication in modern computing. They are used in encryption and decryption processes to ensure that information remains confidential and authentic between communicating parties.

- **Public Key:** This key is shared openly and can be distributed to anyone. It is used to encrypt data that only the corresponding private key can decrypt.
- **Private Key:** This key is kept secret and is only known by the owner. It is used to decrypt data that has been encrypted with the public key.

[Back to TOC](#table-of-contents-toc)

---

### 2. [How Public and Private Keys Work](#how-public-and-private-keys-work)

The public and private keys work together as a pair in cryptographic operations:

- **Encryption with Public Key:**
  - Data is encrypted using the recipient's public key.
  - Once encrypted, the data can only be decrypted by the recipient's private key.
- **Decryption with Private Key:**
  - The recipient uses their private key to decrypt the data.
  - Only the owner of the corresponding private key can decrypt the data, ensuring secure communication.

This process is analogous to a lock-and-key mechanism:
- The public key acts like a lock that anyone can close (encrypt).
- The private key acts like the unique key that can unlock (decrypt) the lock.

[Back to TOC](#table-of-contents-toc)

---

### 3. [Public Key Infrastructure (PKI) and Certificate Authorities (CA)](#public-key-infrastructure-pki-and-certificate-authorities-ca)

Public Key Infrastructure (PKI) is a framework that uses public and private keys to secure communications over networks. PKI relies on trusted entities known as Certificate Authorities (CAs) to issue digital certificates.

- **Certificate Authorities (CA):**
  - A CA is a trusted organization that issues digital certificates.
  - The certificate verifies the ownership of a public key, linking it to an entity like a server or a user.
  - This ensures that when a public key is shared, it truly belongs to the entity it claims to represent.

**Example:**
- When you visit a secure website, your browser checks the website's digital certificate issued by a CA to ensure that the public key you receive truly belongs to that website.

[Back to TOC](#table-of-contents-toc)

---

### 4. [Communication Process Using Public and Private Keys](#communication-process-using-public-and-private-keys)

The communication process using public and private keys involves several steps:

1. **Client Request:**
   - A client sends a request to a server, asking for data or a service.
   
2. **Server Response:**
   - The server responds by sending its public key to the client.
   
3. **Certificate Verification:**
   - The client verifies the server’s public key through a Certificate Authority (CA) to ensure that the key belongs to the legitimate server.
   
4. **Session Key Exchange:**
   - The client generates a session key (a one-time-use symmetric key) and encrypts it with the server's public key.
   - The encrypted session key is then sent to the server.
   
5. **Server Decrypts Session Key:**
   - The server uses its private key to decrypt the session key.
   
6. **Secure Communication:**
   - Both the client and server now share the session key, which they use for secure communication. This key is temporary and specific to that session.

This method ensures that:
- **Confidentiality:** Only the intended recipient (who has the private key) can decrypt the message.
- **Integrity:** The message cannot be tampered with, as any changes would invalidate the encryption.
- **Authentication:** The public key is verified by a trusted CA, confirming the server’s identity.

[Back to TOC](#table-of-contents-toc)
