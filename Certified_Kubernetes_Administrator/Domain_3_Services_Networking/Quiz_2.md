# Domain 3: Services & Networking — Quiz 2 (Ingress & Gateway API)

**Exam:** Certified Kubernetes Administrator (CKA)  
**Format:** 10 Performance Scenario & Multiple Choice Questions with Detailed Explanations

---

### Question 1: Ingress `pathType` (`Prefix` vs `Exact` vs `ImplementationSpecific`)
An administrator creates an Ingress rule with `path: /api/v1` and `pathType: Prefix`. Which of the following request URLs will match this rule, and which will NOT?
- A. `/api/v1` and `/api/v1/users` match; `/api/v10` does NOT match because `Prefix` matches URL path segments split by `/`.
- B. `/api/v1`, `/api/v1/users`, and `/api/v10` all match.
- C. Only `/api/v1` matches.
- D. Only `/api/v1/` with a trailing slash matches.

---

### Question 2: Selecting the Ingress Controller (`ingressClassName`)
A cluster runs two Ingress Controllers (`nginx` and `traefik`). You create an `Ingress` resource without `spec.ingressClassName`, and neither `IngressClass` has the default annotation. What happens?
- A. Both controllers serve the Ingress simultaneously.
- B. Neither controller reconciles the Ingress (`ADDRESS` remains blank) until `spec.ingressClassName: nginx` (or `traefik`) is set on the Ingress, or one `IngressClass` is marked default via `ingressclass.kubernetes.io/is-default-class: "true"`.
- C. `kube-apiserver` rejects the Ingress YAML.
- D. `kube-proxy` serves the Ingress directly.

---

### Question 3: Ingress TLS Termination Secret Type
You need to configure HTTPS TLS termination on an `Ingress` for host `portal.example.com`. What type of Kubernetes `Secret` must be referenced in `spec.tls[].secretName`, and what two data keys must it contain?
- A. `kubernetes.io/tls` containing `tls.crt` and `tls.key` (created via `kubectl create secret tls <name> --cert=tls.crt --key=tls.key`)
- B. `Opaque` containing `cert.pem` and `private.pem`
- C. `kubernetes.io/dockerconfigjson`
- D. `bootstrap.kubernetes.io/token`

---

### Question 4: Gateway API Role-Oriented Resource Hierarchy
The Kubernetes Gateway API (`gateway.networking.k8s.io/v1`) splits Ingress responsibilities across three distinct personas: Infrastructure Provider, Cluster Operator, and Application Developer. Which three core Gateway API resources correspond to those three roles?
- A. `GatewayClass` (Infrastructure Provider), `Gateway` (Cluster Operator), and `HTTPRoute` / `GRPCRoute` / `TLSRoute` (Application Developer)
- B. `IngressClass`, `Ingress`, and `EndpointSlice`
- C. `Service`, `Route`, and `VirtualService`
- D. `NetworkPolicy`, `Gateway`, and `Pod`

---

### Question 5: Binding an `HTTPRoute` to a `Gateway` (`parentRefs`)
In Gateway API, how does an `HTTPRoute` in namespace `app` attach to a `Gateway` named `prod-gateway` in namespace `infra`?
- A. By specifying `spec.parentRefs: [{ name: prod-gateway, namespace: infra }]` in the `HTTPRoute` (and ensuring the `Gateway`'s `listeners[].allowedRoutes.namespaces.from` allows routes from `All` or `Selector` matching `app`).
- B. By setting `spec.gatewayClassName: prod-gateway` on the `HTTPRoute`.
- C. By creating an `ExternalName` Service.
- D. `HTTPRoute` cannot attach to a `Gateway` in a different namespace.

---

### Question 6: Cross-Namespace Backend Reference in Gateway API (`ReferenceGrant`)
An `HTTPRoute` in namespace `frontend` needs to route `/payments` to a `Service` named `payment-svc` located in a different namespace (`billing`). Even after adding `namespace: billing` to `backendRefs`, the route status reports `ResolvedRefs: False (RefNotPermitted)`. Which Gateway API resource must be created in the `billing` namespace to explicitly authorize this cross-namespace reference?
- A. `ReferenceGrant` (allowing `from` group `gateway.networking.k8s.io`, kind `HTTPRoute`, namespace `frontend` `to` group `""`, kind `Service`, name `payment-svc`)
- B. `RoleBinding`
- C. `NetworkPolicy`
- D. `IngressClass`

---

### Question 7: Traffic Splitting / Canary Weights in `HTTPRoute`
You want to perform a 90% / 10% canary traffic split between `web-v1` and `web-v2` Services using the Gateway API without vendor-specific annotations. How is this configured natively in `HTTPRoute`?
- A. Under `spec.rules[].backendRefs`, list both `web-v1` (with `weight: 90`) and `web-v2` (with `weight: 10`).
- B. Create two separate `Gateway` resources.
- C. Use `nginx.ingress.kubernetes.io/canary-weight: "10"`.
- D. Scale `web-v1` to 9 pods and `web-v2` to 1 pod under a single Service.

---

### Question 8: HTTP Header-Based Routing & URL Rewrites in `HTTPRoute`
A CKA task asks you to route requests containing HTTP header `X-Env: staging` to Service `api-staging` and all other requests on `/api` to `api-prod`, while rewriting the path prefix `/api` to `/`. Which `HTTPRoute` features support this natively?
- A. `matches[].headers` (`type: Exact, name: X-Env, value: staging`) and `filters[]` (`type: URLRewrite, urlRewrite: { path: { type: ReplacePrefixMatch, replacePrefixMatch: / } }`)
- B. `spec.tls.passthrough`
- C. `EndpointSlices`
- D. CoreDNS `rewrite` plugin

---

### Question 9: Ingress Default Backend (`defaultBackend`)
In a standard Kubernetes `Ingress` resource, what is the purpose of `spec.defaultBackend`?
- A. It handles all HTTP requests that do not match any `host` or `path` rule defined in `spec.rules`.
- B. It provides DNS resolution for the cluster.
- C. It stores TLS certificates in etcd.
- D. It acts as the default StorageClass.

---

### Question 10: Verifying `Gateway` and `HTTPRoute` Readiness Status
After applying a `Gateway` and `HTTPRoute`, which `status` conditions confirm that the Gateway Controller has provisioned the listener and accepted the route?
- A. On `Gateway`: `Programmed: True` and `Accepted: True`; on `HTTPRoute`: `status.parents[].conditions` showing `Accepted: True` and `ResolvedRefs: True`.
- B. `Phase: Bound`
- C. `Ready: 1/1`
- D. `Endpoints: Active`

---

## Answer Key & Detailed Explanations

1. **A** — `pathType: Prefix` matches based on element-wise `/` path segments; `/api/v10` is a distinct segment from `/api/v1` and does not match.
2. **B** — Modern Ingress controllers ignore `Ingress` resources that lack `spec.ingressClassName` unless a default `IngressClass` is configured in the cluster.
3. **A** — Ingress TLS requires a `kubernetes.io/tls` Secret containing base64-encoded `tls.crt` and `tls.key`.
4. **A** — Gateway API decouples infrastructure class definition (`GatewayClass`), load balancer listener lifecycle (`Gateway`), and application routing rules (`HTTPRoute`).
5. **A** — `HTTPRoute` uses `spec.parentRefs` to bind to one or more `Gateway` listeners, and cross-namespace route attachment is governed by `allowedRoutes.namespaces` on the `Gateway` listener.
6. **A** — `ReferenceGrant` must be created in the target (`billing`) namespace to permit routes from another namespace (`frontend`) to forward traffic to its Services.
7. **A** — `HTTPRoute` supports native `weight` fields inside `backendRefs` without requiring custom controller annotations.
8. **A** — `HTTPRoute` natively standardizes header matching (`matches[].headers`), query parameter matching, HTTP redirects (`RequestRedirect`), header modification (`RequestHeaderModifier`), and path rewrites (`URLRewrite`).
9. **A** — `spec.defaultBackend` is the catch-all backend for unmatched requests reaching the Ingress controller.
10. **A** — Checking `kubectl describe gateway` and `kubectl describe httproute` for `Accepted: True`, `Programmed: True`, and `ResolvedRefs: True` is the standard verification workflow on the CKA exam.
