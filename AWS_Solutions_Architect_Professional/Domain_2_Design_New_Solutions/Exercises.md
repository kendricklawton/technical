# Domain 2: Design for New Solutions — Architecture Exercises & Solutions

## Exercise 1: Multi-Region Active-Active Serverless Architecture with DynamoDB Global Tables
### Architecture Overview
Deploy a globally distributed API across `us-east-1` and `eu-west-1` with Route 53 Latency-based routing, API Gateway, Lambda, and DynamoDB Global Tables.

```mermaid
flowchart TD
    GlobalUsers["Global Clients"] --> R53["Amazon Route 53 (Latency Routing)"]
    R53 -->|Lowest Latency: US| APIGW_US["API Gateway (us-east-1)"]
    R53 -->|Lowest Latency: EU| APIGW_EU["API Gateway (eu-west-1)"]

    APIGW_US --> Lambda_US["AWS Lambda (us-east-1)"]
    APIGW_EU --> Lambda_EU["AWS Lambda (eu-west-1)"]

    Lambda_US --> DDB_US["DynamoDB Table (us-east-1)"]
    Lambda_EU --> DDB_EU["DynamoDB Table (eu-west-1)"]

    DDB_US <==>|Bidirectional Global Table Replication| DDB_EU
```

---

## Exercise 2: Amazon Aurora Global Database Planned Switchover with Zero Data Loss (`RPO = 0`)
1. Create Aurora Global Database with primary in `us-east-1` and secondary in `us-west-2`.
2. Execute managed planned failover via CLI:
   ```bash
   aws rds failover-global-cluster        --global-cluster-identifier enterprise-global-db        --target-db-cluster-identifier arn:aws:rds:us-west-2:111111111111:cluster:enterprise-db-west2
   ```
3. Verify secondary becomes writer and replication reverses automatically.
