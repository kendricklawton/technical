# PE Domain 7: Security, Governance & FinOps — Scenario Quiz 1

Test your knowledge on Workload Identity (SPIFFE/SPIRE, EKS Pod Identity) and Secrets Management.

---

### Question 1
A platform security audit discovers that an application pod authenticates to an AWS S3 bucket using static `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` credentials injected via a Kubernetes Secret. Why is this practice considered an extreme security vulnerability, and how should it be modernized?

A) Static keys expire every 10 minutes.  
B) Static keys do not expire automatically, can be extracted from etcd backups or pod environment variables, and cannot be audited on a per-pod basis. The team should migrate to AWS EKS Pod Identity or IRSA, where pods exchange short-lived tokens for temporary 1-hour IAM credentials automatically.  
C) Kubernetes secrets are always encrypted on developer laptops.  
D) S3 does not support access keys.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Long-lived access keys are the primary source of cloud data breaches when leaked. Workload identity mechanisms (EKS Pod Identity / IRSA) eliminate static keys: the container receives an ephemeral token that is dynamically exchanged for temporary STS credentials valid for 1 hour.
</details>

---

### Question 2
In the SPIFFE/SPIRE architecture, what is the role of **Node Attestation**?

A) It verifies that the server chassis is painted the correct color.  
B) It measures node network bandwidth.  
C) It reboots the Linux kernel.  
D) It securely verifies the identity of the physical or virtual host node (e.g., verifying AWS EC2 instance identity documents or hardware TPM signatures) before the SPIRE Server grants the node's SPIRE Agent permission to request workload identities.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Node Attestation is step 1 of SPIFFE trust establishment. The SPIRE Agent proves its host identity to the SPIRE Server (using cloud instance metadata or hardware TPM). Once the node is authenticated, the agent can attest individual local processes running on that node.
</details>

---

### Question 3
What is the primary advantage of **AWS EKS Pod Identity** over legacy **IRSA (IAM Roles for Service Accounts)**?

A) IRSA requires dedicated bare-metal hardware.  
B) EKS Pod Identity makes all pods cluster administrators.  
C) EKS Pod Identity eliminates the need to configure an external OIDC identity provider in IAM for each cluster, simplifies IAM trust policies across multiple clusters, and operates via a fast local link-local daemon agent.  
D) EKS Pod Identity only works with Python.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** IRSA required creating an IAM OIDC Identity Provider per cluster and authoring complex IAM trust relationships with OIDC sub claims. EKS Pod Identity decouples the cluster from IAM using a cluster association API and a local node agent, dramatically simplifying fleet IAM operations.
</details>

---

### Question 4
When using the **External Secrets Operator (ESO)** in Kubernetes, where does the authoritative, encrypted master secret reside?

A) In the Linux swap partition.  
B) Inside the CoreDNS pod.  
C) In a plaintext file committed to Git.  
D) In an external secure secret manager (e.g., HashiCorp Vault, AWS Secrets Manager, GCP Secret Manager); ESO continuously syncs the values into native Kubernetes Secrets in real time.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** ESO preserves separation of concerns: sensitive credentials are created, rotated, and audited in dedicated external secret managers (Vault/AWS/GCP). ESO retrieves them and creates native Kubernetes Secrets in-cluster, keeping Git completely clean of secret data.
</details>

---

### Question 5
What is the **Transit Secrets Engine** in HashiCorp Vault?

A) A "Cryptography-as-a-Service" engine where applications send plaintext to Vault over an API, Vault encrypts or decrypts the data using an internal master key, and returns the result without the application ever having access to the encryption key itself.  
B) A tool that converts JSON to XML.  
C) A system for booking train tickets.  
D) A high-speed network proxy.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** The Transit engine handles cryptography offload. Applications do not need to store encryption keys or manage key rotation algorithms; they submit data to Vault's transit API, which handles encryption, decryption, and HMAC generation securely.
</details>

---

### Question 6
What is the function of **Shamir's Secret Sharing** algorithm in HashiCorp Vault initialization?

A) It splits the Vault master unseal key into $N$ distinct shards such that any $M$ threshold of shards (e.g., 3 of 5) must be entered to unseal Vault, preventing any single rogue employee from compromising or decrypting the entire database.  
B) It generates random passwords for developers.  
C) It compresses data blocks on disk.  
D) It routes network packets between Availability Zones.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Shamir's algorithm provides threshold cryptography. Vault's master key is split among key custodians. Unsealing Vault after a reboot requires a quorum of key holders to enter their shards, ensuring no single administrator can unilaterally compromise the secret vault.
</details>

---

### Question 7
In SPIFFE, what is an **SVID**?

A) A database table primary key.  
B) A special video file format.  
C) A SPIFFE Verifiable Identity Document: a cryptographically signed x509 certificate or JWT token that encodes the workload's SPIFFE ID and is used for authentication during mTLS handshakes.  
D) An Amazon EC2 instance type.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** An SVID is the tangible cryptographic credential representing a SPIFFE ID. It is presented by a workload during connection establishment to prove its identity to peers.
</details>

---

### Question 8
What happens if the External Secrets Operator controller crashes or loses network connectivity to AWS Secrets Manager?

A) The API server rolls back to the previous version.  
B) Pre-existing Kubernetes Secrets already created by ESO remain active and accessible in the cluster, ensuring that running workloads experience zero downtime while external connectivity is restored.  
C) etcd deletes all cluster data.  
D) All running application pods crash immediately.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** ESO creates native Kubernetes `Secret` resources in etcd. Running pods read from these local secrets. If ESO crashes or the external cloud provider experiences an API outage, existing pods continue serving traffic unaffected.
</details>

---

### Question 9
What is a **Dynamic Secret** in HashiCorp Vault?

A) A secret that is shared publicly on Slack.  
B) An unencrypted environment variable.  
C) A secret that changes its name every minute.  
D) A credential that Vault generates on demand (e.g., creating a unique PostgreSQL user account with a 1-hour lease) specifically for a requesting client, automatically destroying the account when the lease expires.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Dynamic secrets eliminate shared passwords. Instead of sharing a single `admin` password across 50 services, Vault dynamically provisions a unique, temporary database credential per service and automatically drops the role from the database upon lease expiration.
</details>

---

### Question 10
Why is storing sensitive secrets directly in container images (e.g., `ENV DB_PASSWORD=secret` in a Dockerfile) an extreme security violation?

A) Environment variables and Dockerfile build instructions are permanently baked into the image metadata and layer history, making them visible to anyone with read access to the container registry via `docker inspect` or `docker history`.  
B) Docker will refuse to build the image.  
C) It slows down container boot time by 5 minutes.  
D) It prevents the container from writing logs.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Hardcoded image environment variables are completely unencrypted. Anyone with pull access to the registry can execute `docker history --no-trunc` and read the plaintext password directly from the metadata.
</details>
