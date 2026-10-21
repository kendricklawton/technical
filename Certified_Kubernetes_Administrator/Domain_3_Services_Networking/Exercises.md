# Domain 3: Services & Networking — Hands-On Exercises & Solutions

## Exercise 1: Multi-Tier NetworkPolicy with DNS Egress and Specific Namespace Isolation
### Objective
Isolate an application in namespace `finance` such that:
1. Only pods labeled `role=api-client` in namespace `payment` can access it on port `8443`.
2. Outbound egress is strictly forbidden EXCEPT for DNS resolution to `kube-system` on port 53 (UDP & TCP).

### Complete Manifest (`finance-netpol.yaml`)
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: isolate-finance-api
  namespace: finance
spec:
  podSelector:
    matchLabels:
      app: finance-vault
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: payment
      podSelector:
        matchLabels:
          role: api-client
    ports:
    - protocol: TCP
      port: 8443
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: kube-system
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```

### Verification
```bash
kubectl apply -f finance-netpol.yaml
# Test connection from non-matching pod (must fail with timeout)
kubectl run test-fail -n payment --image=busybox:1.28 --rm -it -- restart=Never -- nc -zvw3 finance-vault.finance 8443
```

---

## Exercise 2: NGINX Ingress Controller with Path-Based Routing & TLS Termination
### Objective
Expose `order-service:8080` under path `/orders` and `customer-service:8081` under path `/customers` with TLS certificate `api-tls-cert`.

### Solution
```bash
# Generate TLS secret
openssl req -x509 -nodes -days 365 -newkey rsa:2048   -keyout tls.key -out tls.crt -subj "/CN=shop.example.com"
kubectl create secret tls api-tls-cert -n ecommerce --key tls.key --cert tls.crt
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: shop-ingress
  namespace: ecommerce
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - shop.example.com
    secretName: api-tls-cert
  rules:
  - host: shop.example.com
    http:
      paths:
      - path: /orders(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: order-service
            port:
              number: 8080
      - path: /customers(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: customer-service
            port:
              number: 8081
```

---

## Exercise 3: CoreDNS Custom Forwarding & Stub Domains
### Objective
Configure CoreDNS to forward queries for domain `corp.internal` to DNS server `10.0.0.53:53`.

### Solution
Edit ConfigMap `coredns` in `kube-system`:
```bash
kubectl edit cm coredns -n kube-system
```
Add server block:
```text
corp.internal:53 {
    errors
    cache 30
    forward . 10.0.0.53
}
```
Restart CoreDNS pods:
```bash
kubectl rollout restart deployment/coredns -n kube-system
```

---

## Exercise 4: Gateway API HTTPRoute with Canary Traffic Splitting
### Objective
Deploy an `HTTPRoute` splitting traffic 80% to `web-v1` and 20% to `web-v2-canary`.

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: web-route
  namespace: production
spec:
  parentRefs:
  - name: shared-gateway
  hostnames:
  - "app.example.com"
  rules:
  - backendRefs:
    - name: web-v1
      port: 80
      weight: 80
    - name: web-v2-canary
      port: 80
      weight: 20
```

---

## Exercise 5: Troubleshooting Service Selector & Endpoint Mismatches
### Objective
A service `analytics-svc` returns `503 Service Unavailable` or `connection refused`. Identify and fix the selector mismatch.

```bash
# Step 1: Check service endpoints
kubectl get endpoints analytics-svc
# If <none>, examine service selector
kubectl get svc analytics-svc -o yaml | grep -A 3 selector

# Step 2: Compare with pod labels
kubectl get pods --show-labels

# Step 3: Patch selector
kubectl patch svc analytics-svc -p '{"spec":{"selector":{"app":"analytics","tier":"backend"}}}'
```

---

## Exercise 6: Headless Service for StatefulSet Discovery
### Objective
Create a Headless service `db-headless` for a StatefulSet `db-cluster` in namespace `databases`.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: db-headless
  namespace: databases
spec:
  clusterIP: None
  selector:
    app: db-cluster
  ports:
  - port: 5432
    name: postgresql
```
