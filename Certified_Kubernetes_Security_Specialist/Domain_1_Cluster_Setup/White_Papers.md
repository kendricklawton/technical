# Domain 1: Cluster Setup — Essential Security Benchmarks & Technical Documentation Guide

> **Blueprint Weight:** 10% of Certified Kubernetes Security Specialist (CKS)  
> **Core Objective:** Implement CIS Kubernetes Benchmark recommendations, secure network access to the API server, enforce ingress TLS termination, and isolate cluster infrastructure using NetworkPolicies.

---

## 1. Authoritative Security Standards & Documentation

### 1.1 [CIS Kubernetes Benchmark](https://www.cisecurity.org/benchmark/kubernetes)
* **Canonical Scope:** Consensus-driven security configuration baseline for Kubernetes components.
* **Control Plane Hardening Parameters (`/etc/kubernetes/manifests/`):**
  * `kube-apiserver`:
    * `--anonymous-auth=false` (Block unauthenticated anonymous requests).
    * `--profiling=false` (Prevent profiling memory dump leaks).
    * `--enable-admission-plugins=...,NodeRestriction,...`
    * `--authorization-mode=Node,RBAC` (Disable `AlwaysAllow`).
  * `kube-controller-manager`:
    * `--use-service-account-credentials=true`
  * `etcd`:
    * `--client-cert-auth=true`, `--auto-tls=false` (Enforce mTLS).

### 1.2 [NSA / CISA Kubernetes Cybersecurity Technical Report](https://www.cisa.gov/resources-reference/resources/kubernetes-cybersecurity-guidance)
* **Canonical Scope:** Architectural guidance for zero trust Kubernetes deployment.
* **Core Recommendations:**
  * Network separation of control plane and worker nodes.
  * Restricting control plane API access to administrative bastion jumpboxes.
  * Encrypting all pod-to-pod and control plane traffic using WireGuard or IPsec CNI encryption.

### 1.3 [Controlling Access to the Kubernetes API](https://kubernetes.io/docs/concepts/security/controlling-access/)
* **Canonical Scope:** The 3-stage authentication, authorization, and admission control pipeline.
* **Pipeline Sequence:**
  1. **Authentication:** TLS Certificates, Bearer Tokens, Webhook Tokens.
  2. **Authorization:** Node, RBAC, Webhook.
  3. **Admission Control:** Mutating webhooks -> Schema validation -> Validating webhooks / ValidatingAdmissionPolicy.

---

## 2. High-Frequency Exam Traps & Practical Reflexes

1. **`kube-bench` Audit & Remediation:**
   * Run `kube-bench run --targets master` or `node`.
   * For any `[FAIL]` item, navigate to `/etc/kubernetes/manifests/` and insert/correct the missing command-line argument. The kubelet detects manifest changes and restarts the static pod automatically.
2. **NetworkPolicy Namespace Boundary Isolation:**
   * Restricting ingress traffic to a database pod only from specific microservice pods in another namespace requires BOTH `namespaceSelector` AND `podSelector`:
     ```yaml
     ingress:
     - from:
       - namespaceSelector:
           matchLabels:
             kubernetes.io/metadata.name: frontend
         podSelector:
           matchLabels:
             app: web
     ```
   * *Trap:* Listing them as separate array items creates an `OR` condition instead of an `AND` condition!

---

## 3. Reading & Study Checklist

- [ ] Read *CIS Kubernetes Benchmark (Master & Node sections)*
- [ ] Read *NSA / CISA Kubernetes Cybersecurity Technical Report*
- [ ] Read *Controlling Access to the Kubernetes API*
- [ ] Practice running and remediating findings from `kube-bench`
- [ ] Master multi-selector NetworkPolicies combining namespaces and pods
