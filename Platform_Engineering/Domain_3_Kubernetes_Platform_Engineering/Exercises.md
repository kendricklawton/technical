# PE Domain 3: Kubernetes Platform Engineering — Hands-On Labs

Six production-grade labs drilling CRDs, controller reconciliation, in-tree admission policies, vCluster multi-tenancy, Karpenter autoscaling, and Gateway API routing.

---

## Lab 1: Building a Custom Resource Definition with In-Tree CEL Validation Rules

### Objective
Define an enterprise `PostgresDatabase` CRD featuring schema validation, `/status` subresource, and in-tree Common Expression Language (CEL) validation rules that enforce architectural policies at the API server level.

### Step-by-Step Implementation

1. **Write the CRD Manifest (`postgres-crd.yaml`):**
   ```yaml
   apiVersion: apiextensions.k8s.io/v1
   kind: CustomResourceDefinition
   metadata:
     name: postgresdatabases.platform.example.com
   spec:
     group: platform.example.com
     names:
       kind: PostgresDatabase
       plural: postgresdatabases
       singular: postgresdatabase
       shortNames:
         - pgdb
     scope: Namespaced
     versions:
       - name: v1alpha1
         served: true
         storage: true
         subresources:
           status: {}
         schema:
           openAPIV3Schema:
             type: object
             properties:
               spec:
                 type: object
                 required:
                   - storageGb
                   - engineVersion
                   - environment
                 properties:
                   storageGb:
                     type: integer
                     minimum: 10
                     maximum: 5000
                   engineVersion:
                     type: string
                     enum: ["14", "15", "16"]
                   environment:
                     type: string
                     enum: ["development", "staging", "production"]
                   highAvailability:
                     type: boolean
                     default: false
                 # CEL Validation Rules
                 x-kubernetes-validations:
                   - rule: "self.environment != 'production' || self.highAvailability == true"
                     message: "Production databases MUST enable highAvailability."
                   - rule: "self.environment != 'production' || self.storageGb >= 50"
                     message: "Production databases must allocate at least 50Gb storage."
               status:
                 type: object
                 properties:
                   endpoint:
                     type: string
                   phase:
                     type: string
   ```

2. **Apply and Test Validation:**
   ```bash
   kubectl apply -f postgres-crd.yaml
   
   # Attempt to apply a non-compliant production database (should FAIL)
   cat <<EOF | kubectl apply -f -
   apiVersion: platform.example.com/v1alpha1
   kind: PostgresDatabase
   metadata:
     name: bad-prod-db
   spec:
     storageGb: 20
     engineVersion: "16"
     environment: "production"
     highAvailability: false
   EOF
   ```

Observe the API server rejection message: `Production databases MUST enable highAvailability`.

---

## Lab 2: Scaffold and Run a Controller Reconciler Loop

### Objective
Write an idempotent reconciliation function in Go simulating a custom Kubernetes controller that monitors `PostgresDatabase` resources, manages external state, and updates the `.status` subresource.

### Implementation (`reconciler.go`)
```go
package main

import (
	"context"
	"fmt"
	"time"
)

type DatabaseSpec struct {
	StorageGb        int
	EngineVersion    string
	Environment      string
	HighAvailability bool
}

type DatabaseStatus struct {
	Phase    string
	Endpoint string
}

type PostgresDatabase struct {
	Namespace string
	Name      string
	Spec      DatabaseSpec
	Status    DatabaseStatus
}

// Reconcile simulates an idempotent controller loop
func Reconcile(ctx context.Context, db *PostgresDatabase) error {
	fmt.Printf("[Reconcile] Processing %s/%s\n", db.Namespace, db.Name)

	if db.Status.Phase == "Ready" {
		fmt.Println("[Reconcile] Resource already in desired state. Nothing to do.")
		return nil
	}

	fmt.Printf("[Reconcile] Provisioning AWS RDS Postgres %s in %s environment...\n",
		db.Spec.EngineVersion, db.Spec.Environment)
	time.Sleep(500 * time.Millisecond) // Simulate cloud API latency

	// Update status
	db.Status.Phase = "Ready"
	db.Status.Endpoint = fmt.Sprintf("%s.rds.internal:5432", db.Name)
	fmt.Printf("[Reconcile] Successfully provisioned! Endpoint: %s\n", db.Status.Endpoint)
	return nil
}

func main() {
	db := &PostgresDatabase{
		Namespace: "checkout",
		Name:      "orders-db",
		Spec: DatabaseSpec{
			StorageGb:        100,
			EngineVersion:    "16",
			Environment:      "production",
			HighAvailability: true,
		},
		Status: DatabaseStatus{Phase: "Pending"},
	}

	// First reconcile: provisions resource
	Reconcile(context.Background(), db)

	// Second reconcile: idempotent check
	Reconcile(context.Background(), db)
}
```

### Verification
```bash
go run reconciler.go
```

---

## Lab 3: Enforcing In-Tree Security Guardrails Using `ValidatingAdmissionPolicy`

### Objective
Deploy a native `ValidatingAdmissionPolicy` in Kubernetes 1.30+ to forbid containers from running with root privileges or missing CPU/memory limits.

### Step-by-Step Implementation

1. **Define the Policy (`security-policy.yaml`):**
   ```yaml
   apiVersion: admissionregistration.k8s.io/v1
   kind: ValidatingAdmissionPolicy
   metadata:
     name: enforce-container-security
   spec:
     failurePolicy: Fail
     matchConstraints:
       resourceRules:
         - apiGroups: [""]
           apiVersions: ["v1"]
           operations: ["CREATE", "UPDATE"]
           resources: ["pods"]
     validations:
       - expression: "object.spec.containers.all(c, has(c.resources) && has(c.resources.limits) && has(c.resources.limits.memory))"
         message: "All containers must define a memory limit."
       - expression: "object.spec.containers.all(c, has(c.securityContext) && has(c.securityContext.runAsNonRoot) && c.securityContext.runAsNonRoot == true)"
         message: "All containers must set securityContext.runAsNonRoot to true."
   ---
   apiVersion: admissionregistration.k8s.io/v1
   kind: ValidatingAdmissionPolicyBinding
   metadata:
     name: enforce-container-security-binding
   spec:
     policyName: enforce-container-security
     validationActions: [Deny]
     matchResources:
       namespaceSelector:
         matchExpressions:
           - key: kubernetes.io/metadata.name
             operator: NotIn
             values: ["kube-system", "kube-node-lease"]
   ```

2. **Apply Policy:**
   ```bash
   kubectl apply -f security-policy.yaml
   ```

---

## Lab 4: Multi-Tenant Isolation with vCluster

### Objective
Deploy a Virtual Cluster (`vCluster`) on a host cluster to provide a tenant with dedicated control plane APIs, CRD independence, and full admin rights without cluster-level blast radius.

### Step-by-Step Implementation

1. **Deploy a vCluster using Helm:**
   ```bash
   helm repo add loft-sh https://charts.loft.sh
   helm repo update
   helm install team-alpha-vcluster loft-sh/vcluster --namespace team-alpha --create-namespace
   ```

2. **Access the Virtual Cluster:**
   ```bash
   # Connect to vcluster context
   vcluster connect team-alpha-vcluster -n team-alpha --update-current=false --kubeconfig /tmp/vcluster.kubeconfig
   
   # Verify isolated API server
   kubectl --kubeconfig=/tmp/vcluster.kubeconfig get namespaces
   ```

3. **Verify Host Isolation:**
   Create a Pod inside the vCluster. Observe that on the host cluster, the pod runs inside `team-alpha` namespace with a synthesized name, sharing the host worker node kernel safely.

---

## Lab 5: Dynamic Autoscaling with KEDA

### Objective
Deploy KEDA to scale a worker deployment from **0 to 10 replicas** based on simulated Prometheus queue metrics.

### Step-by-Step Implementation

1. **Define the ScaledObject (`scaled-worker.yaml`):**
   ```yaml
   apiVersion: keda.sh/v1alpha1
   kind: ScaledObject
   metadata:
     name: order-processor-scaler
     namespace: default
   spec:
     scaleTargetRef:
       name: order-processor
     minReplicaCount: 0
     maxReplicaCount: 10
     cooldownPeriod: 30
     triggers:
       - type: prometheus
         metadata:
           serverAddress: http://prometheus-k8s.monitoring.svc:9090
           metricName: pending_orders_total
           query: sum(pending_orders_total)
           threshold: '50'
   ```

---

## Lab 6: Traffic Splitting with Gateway API & HTTPRoute

### Objective
Configure a Gateway API `HTTPRoute` to execute an automated 90/10 canary split between `v1` and `v2` backend services.

### Implementation Manifest (`canary-httproute.yaml`)
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: payments-route
  namespace: payments
spec:
  parentRefs:
    - name: shared-gateway
      namespace: platform-system
  hostnames:
    - "payments.example.com"
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /v1/charge
      backendRefs:
        - name: payments-v1
          port: 8080
          weight: 90
        - name: payments-v2
          port: 8080
          weight: 10
```
