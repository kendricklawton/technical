# Domain 3: Continuous Improvement — Architecture Exercises & Solutions

## Exercise 1: Mitigating Lambda Connection Exhaustion with Amazon RDS Proxy
Deploy an Amazon RDS Proxy between a fleet of serverless Lambda functions and an Aurora PostgreSQL DB cluster to pool and multiplex database connections.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  DatabaseProxy:
    Type: AWS::RDS::DBProxy
    Properties:
      DBProxyName: payment-db-proxy
      EngineFamily: POSTGRESQL
      RoleArn: !GetAtt ProxyIAMRole.Arn
      VpcSubnetIds:
        - subnet-0123456789abcdef0
        - subnet-0fedcba9876543210
      Auth:
        - AuthScheme: SECRETS
          SecretArn: arn:aws:secretsmanager:us-east-1:111111111111:secret:db-credentials
          IAMAuth: REQUIRED
```
