# Domain 1: Design for Organizational Complexity — Architecture Exercises & Solutions

## Exercise 1: Hybrid Networking with AWS Direct Connect, Transit Gateway & Active/Passive Failover
Design a high-availability Direct Connect architecture connecting on-premises data centers to a multi-account Transit Gateway with IPsec VPN backup.

```mermaid
flowchart TD
    OnPrem["Corporate On-Premises Data Center"] -->|Primary: 10 Gbps DX| DXGW["Direct Connect Gateway"]
    OnPrem -->|Backup: IPsec VPN over Internet| TGW_VPN["TGW VPN Attachment"]
    DXGW -->|Transit VIF| TGW["AWS Transit Gateway"]
    TGW --> VPC_A["Spoke VPC A (Workloads)"]
    TGW --> VPC_B["Spoke VPC B (Shared Services)"]
    TGW --> Egress["Centralized Egress VPC"]
```

### Routing Precedence Configuration
1. Direct Connect advertises AWS prefixes via BGP.
2. On-premises advertises corporate prefixes with AS-PATH prepending on the backup VPN link (e.g., 3 extra AS hops) or by tagging BGP communities (`7224:7100` for low preference, `7224:7300` for high preference).
3. Transit Gateway route table installs Direct Connect routes over VPN because DX routes have higher priority when prefix lengths are equal.
