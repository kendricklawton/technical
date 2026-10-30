# Domain 3: Designing for Security, Identity, and Compliance — Architectural White Papers

---

## White Paper 1: Zero Trust Security with BeyondCorp Enterprise and Identity-Aware Proxy

### Abstract
Traditional perimeter security assumes that anyone inside the corporate network is trusted. BeyondCorp shifts access controls from network perimeters to individual users and devices, enabling secure access to internal enterprise workloads from any location without a VPN.

### Architecture Topology
```
[ Remote Employee / Contractor ]
               │
               ▼ HTTPS (Internet)
[ Google Global Edge Point of Presence (PoP) ]
               │
               ▼
[ BeyondCorp Identity-Aware Proxy (IAP) ]
   ├── Evaluates Context-Aware Access Policy:
   │     1. User Identity (Google Workspace / Cloud Identity MFA)
   │     2. Device Health (Managed corporate device, OS version)
   │     3. Geolocation (Must originate from approved country)
   │     4. IP Subnet / Endpoint Verification
               │
      [ Policy Passed ]
               │
               ▼ Internal RFC 1918 Routing
[ Internal HTTP(S) Load Balancer / GKE Pods / Compute VMs ]
(Instances have ZERO public external IP addresses)
```

---

## White Paper 2: Data Exfiltration Defense via VPC Service Controls (VPC-SC)

### Abstract
Malicious insiders or compromised credentials can use legitimate API tokens to copy sensitive datasets from an enterprise Cloud Storage bucket to a personal storage bucket outside the company. VPC Service Controls provides a cryptographic perimeter around Google Cloud APIs to prevent unauthorized egress.

### Perimeter Mechanics
```
                      ENTERPRISE SECURE PERIMETER
 ┌──────────────────────────────────────────────────────────────────┐
 │                                                                  │
 │   [ Compute Engine VM / GKE Pod ]                                │
 │   (Project: analytics-prod)                                      │
 │                 │                                                │
 │                 ▼ Internal Google API Call                       │
 │   [ Sensitive BigQuery / Cloud Storage ]                         │
 │                                                                  │
 └─────────────────┬────────────────────────────────────────────────┘
                   │
                   ▼ ATTEMPT: Copy data to external bucket
 ┌─────────────────┴────────────────────────────────────────────────┐
 │                 X BLOCKED BY VPC SERVICE CONTROLS                │
 │ "Request denied: Destination resource outside security perimeter"│
 └──────────────────────────────────────────────────────────────────┘
```
