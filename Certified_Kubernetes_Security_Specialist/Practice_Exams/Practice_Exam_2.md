# CKS Mock Practice Exam 2

**Duration:** 120 Minutes  
**Passing Score:** 67%  
**Format:** 16 Hands-on Security Tasks  

---

### Task 1: etcd Secret Encryption at Rest (Weight: 8%)
Configure secret encryption using AES-CBC with key `k8s-secret-encryption-key-32bytes!` and verify that existing secrets are re-encrypted.

---

### Task 2: Restrict Container Privileges with PSA (Weight: 6%)
Label namespace `secure-production` to enforce the `restricted` Pod Security Standard.

---

### Task 3: Runtime Threat Detection with Falco (Weight: 7%)
Add a custom rule to `/etc/falco/falco_rules.local.yaml` to alert at `CRITICAL` priority when a sensitive file `/etc/shadow` is opened for reading inside any container.
