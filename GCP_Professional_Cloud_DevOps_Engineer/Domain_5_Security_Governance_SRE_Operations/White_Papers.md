# Domain 5: Security, Governance & SRE Operations — Architectural White Papers

---

## White Paper 1: Zero-Key CI/CD Pipeline Architecture via Workload Identity Federation

### Abstract
Eliminating static cryptographic keys is the single most impactful security posture improvement for enterprise DevOps. This paper presents the architecture, security invariants, and OIDC token exchange mechanics connecting external CI systems (GitHub Actions) to Google Cloud without storing long-lived service account keys.

### Architecture Topology
```
[ GitHub Actions Runner ]
       │
       │ 1. Requests OIDC JWT (audience: gcp-wif-pool)
       ▼
[ GitHub OIDC IdP (token.actions.githubusercontent.com) ]
       │
       │ 2. Issues Signed JWT with claims (sub: repo:org/app:ref:refs/heads/main)
       ▼
[ Google Cloud STS (Security Token Service) ]
       │
       │ 3. Validates OIDC Signature & Attribute Condition Mapping
       ▼
[ IAM Workload Identity Pool / Provider ]
       │
       │ 4. Issues Federate Token (gcp-token)
       ▼
[ IAM Service Account Credentials API ]
       │
       │ 5. Impersonates GSA: cicd-deployer@project.iam.gserviceaccount.com
       ▼
[ Target Google Cloud APIs (Cloud Deploy, Artifact Registry, GKE) ]
```

### Security Invariants & Attribute Restrictions
To prevent arbitrary GitHub repositories from impersonating the deployment service account, an explicit attribute condition MUST be enforced:
```
attribute.repository == "acme-corp/payment-service" &&
attribute.ref == "refs/heads/main"
```

---

## White Paper 2: Defense-in-Depth Container Supply Chain with Binary Authorization

### Abstract
Modern software delivery requires cryptographic guarantees that only source code built by authenticated CI pipelines can execute in production. This architecture integrates Cloud Build, Container Analysis, Cloud KMS, and GKE Binary Authorization.

### Enforcement Workflow
1. **Source Attestation:** Developer signs commit via GPG.
2. **Build Provenance:** Cloud Build produces container image and generates SLSA Level 3 provenance metadata stored in Container Analysis.
3. **Vulnerability Scanning:** Automated Vulnerability Scanning verifies image has zero CVEs with severity `CRITICAL` or `HIGH`.
4. **KMS Signature:** An automated Cloud Function reviews scan results; if clean, signs an attestation note using an asymmetric Cloud KMS signing key.
5. **GKE Admission Webhook:** GKE Binary Authorization admission controller intercepts pod creation. If attestation is absent or signature fails KMS validation, the pod is rejected.
