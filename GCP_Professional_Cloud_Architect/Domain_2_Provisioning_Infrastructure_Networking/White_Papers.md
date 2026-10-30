# Domain 2: Managing and Provisioning Infrastructure & Networking — Architectural White Papers

---

## White Paper 1: Enterprise Multi-Region Network Topology with Shared VPC & Dedicated Interconnect

### Abstract
Large enterprise architectures require strict separation of duties between central network infrastructure teams and distributed application delivery teams. This paper outlines the architecture of an enterprise landing zone spanning multiple regions with Dedicated Cloud Interconnect and Shared VPC.

### Topology Architecture
```
[ Corporate On-Premises Data Centers ]
         │ (Primary Metro)             │ (Secondary Metro)
         ▼ 100 Gbps Dedicated Interconnect
[ Google Cloud Edge Routers (Cloud Routers with Dynamic BGP) ]
         │
═════════╪═══════════════════════════════════════════════════════
         │ Host Project: "corp-network-hub"
         ▼
[ Enterprise Shared VPC (Global Network: 10.0.0.0/8) ]
   ├── Regional Subnet US: 10.10.0.0/20 (us-central1)
   └── Regional Subnet EU: 10.20.0.0/20 (europe-west1)
         │
    Attached Service Projects (Least-Privilege RBAC)
         ├───────────────────────────────┐
         ▼                               ▼
Service Project: "billing-prod"   Service Project: "crm-prod"
- Regional GKE Autopilot Cluster  - Regional Compute Engine MIG
- Attached to Subnet US           - Attached to Subnet EU
```

### Security Invariants & Isolation
1. **Network Admin Separation:** Application teams in Service Projects hold `Compute Instance Admin` but zero permissions to modify routes, subnets, or firewall rules.
2. **Dynamic BGP Failover:** Cloud Router advertises VPC subnets via BGP MED (Multi-Exit Discriminator) attributes to ensure symmetric routing during active-passive data center failovers.

---

## White Paper 2: Anycast Global Load Balancing with Cloud Armor and Cloud CDN

### Abstract
Traditional DNS-based multi-region routing suffers from high client-side TTL caching delays during regional failovers. Google Cloud's Global External HTTP(S) Load Balancer utilizes Anycast BGP routing to terminate client TCP/TLS handshakes at the edge PoP, enabling sub-second multi-region traffic migration.

### Packet Flow & Edge Acceleration
```
Client (Frankfurt) ───► [ Google Edge PoP (Frankfurt) ]
                         - Anycast IP Resolution (Single Global VIP)
                         - TCP / TLS Handshake Termination
                         - Cloud Armor OWASP WAF Inspection
                         - Cloud CDN Edge Cache Lookup (HIT -> Return)
                               │
                               ▼ (MISS -> Google Global Private Fiber)
                         [ Backend Service: GKE Cluster (europe-west1) ]
                               │ (Overloaded / Outage)
                               ▼ (Automatic Sub-second Spillover)
                         [ Backend Service: GKE Cluster (us-central1) ]
```
