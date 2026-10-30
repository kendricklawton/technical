# Domain 2: Self-Service Platform APIs & Automation — Performance Labs & Drills

---

## Lab 1: Building a Complete Crossplane XRD and Composition

### Objective
Define an enterprise CompositeResourceDefinition (XRD) for an object storage bucket (`XBucket`), author a Composition mapping it to a cloud provider bucket, and claim the bucket using a developer-facing `BucketClaim`.

```yaml
# 1. CompositeResourceDefinition (xbuckets.storage.platform.io)
apiVersion: apiextensions.crossplane.io/v1
kind: CompositeResourceDefinition
metadata:
  name: xbuckets.storage.platform.io
spec:
  group: storage.platform.io
  names:
    kind: XBucket
    plural: xbuckets
  claimNames:
    kind: BucketClaim
    plural: bucketclaims
  versions:
    - name: v1alpha1
      served: true
      referenceable: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              required: [region]
              properties:
                region:
                  type: string
                versioning:
                  type: boolean
                  default: true
---
# 2. Composition (xbucket.digitalocean.spaces)
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  name: xbucket.digitalocean
  labels:
    provider: digitalocean
spec:
  compositeTypeRef:
    apiVersion: storage.platform.io/v1alpha1
    kind: XBucket
  resources:
    - name: do-spaces-bucket
      base:
        apiVersion: storage.do.crossplane.io/v1alpha1
        kind: SpacesBucket
        spec:
          forProvider:
            region: nyc3
            acl: private
      patches:
        - type: FromCompositeFieldPath
          fromFieldPath: spec.region
          toFieldPath: spec.forProvider.region
---
# 3. Developer Claim (claim.yaml)
apiVersion: storage.platform.io/v1alpha1
kind: BucketClaim
metadata:
  name: analytics-raw-data
  namespace: analytics
spec:
  region: nyc3
  versioning: true
```

---

## Lab 2: Authoring a Backstage Software Template for Self-Service Deployments

### Objective
Create a Backstage Software Template that collects developer parameters (app name, port, team), renders a skeleton directory with a Dockerfile and Kubernetes manifests, and pushes the new repository to GitHub.
