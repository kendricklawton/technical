# Domain 5: Supply Chain Security — Hands-On Exercises & Solutions

## Exercise 1: Scan Container Image with Trivy and Fail Pipeline
```bash
# Scan image and exit with 1 if CRITICAL or HIGH vulnerabilities exist
trivy image --severity HIGH,CRITICAL --exit-code 1 --ignore-unfixed alpine:3.14
```

## Exercise 2: Generate Software Bill of Materials (SBOM) in SPDX Format
```bash
trivy image --format spdx-json --output /tmp/nginx-sbom.json nginx:latest
```

## Exercise 3: Sign Container Image with Cosign
```bash
cosign generate-key-pair
cosign sign --key cosign.key myrepo/myimage:v1.0.0
cosign verify --key cosign.pub myrepo/myimage:v1.0.0
```
