# Domain 1: GitOps & Continuous Delivery — Performance Labs & Drills

---

## Lab 1: Multi-Tenant ApplicationSet Matrix Generator Setup

### Objective
Deploy an Argo CD `ApplicationSet` using a Matrix Generator combining:
1. A Git generator scanning directories in `apps/*`.
2. A Cluster generator matching target clusters labeled `env=staging`.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: multi-tenant-matrix
  namespace: argocd
spec:
  generators:
    - matrix:
        generators:
          - clusters:
              selector:
                matchLabels:
                  env: staging
          - git:
              repoURL: https://github.com/enterprise/platform-apps.git
              revision: HEAD
              directories:
                - path: apps/*
  template:
    metadata:
      name: '{{name}}-{{path.basename}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/enterprise/platform-apps.git
        targetRevision: HEAD
        path: '{{path}}'
      destination:
        server: '{{server}}'
        namespace: '{{path.basename}}'
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
        syncOptions:
          - CreateNamespace=true
```

---

## Lab 2: Automated Rollout Canary Deployment with Prometheus Analysis

### Objective
Configure an Argo Rollout that promotes a new container version in 3 steps (20%, 50%, 100%) while executing an automated Prometheus `AnalysisTemplate` asserting that HTTP 5xx error rate remains below 1%.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: error-rate-analysis
  namespace: default
spec:
  metrics:
    - name: http-5xx-errors
      interval: 30s
      count: 3
      failureLimit: 1
      successCondition: result[0] < 0.01
      provider:
        prometheus:
          address: http://prometheus-k8s.monitoring.svc:9090
          query: |
            sum(rate(http_requests_total{status=~"5.*"}[1m]))
            /
            sum(rate(http_requests_total[1m]))
```
