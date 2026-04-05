# Infrastructure — Terraform
## Project Kronos Multi-Cloud IaC

This directory contains Terraform configuration references for the Project Kronos multi-cloud environment.

> ⚠️ **Note:** Actual `.tfstate` files and credential values are NOT stored in this repository. See `.gitignore`. Terraform state should be stored in a backend with encryption at rest and access logging enabled.

---

## Environment Summary

| Cloud | Resources Deployed | Free Tier |
|-------|-------------------|-----------|
| AWS | EC2 (t2.micro), S3 buckets, VPC, VPN Gateway | ✅ Yes |
| Azure | VMs (B1s), VNet, Public IPs, NSGs | ✅ Yes |
| GCP | Compute Engine (e2-micro), Cloud Storage, VPC | ✅ Yes |

---

## Key Lesson — State File Security

**Critical finding from Project Kronos simulation:**

Terraform state files stored in unprotected S3 buckets can expose credentials for all managed cloud environments. The `azure_sp.json` artifact in S3 bucket `cloud-lab-states` directly enabled Stage 2 of the attack chain.

**Required controls for Terraform state:**

```hcl
# Example: Secure S3 backend for Terraform state
terraform {
  backend "s3" {
    bucket         = "terraform-state-bucket"
    key            = "project-kronos/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true          # AES-256 encryption at rest
    kms_key_id     = "arn:aws:kms:..."
    dynamodb_table = "terraform-state-lock"

    # Access logging must be enabled on this bucket
    # Bucket must NOT be public
    # Versioning must be enabled
  }
}
```

---

## Security Requirements for Terraform Operations

1. Never use long-lived static credentials in Terraform — use IAM roles or managed identities
2. Always encrypt Terraform state at rest
3. Enable access logging on state storage buckets
4. Use pre-commit hooks for secret scanning before any `terraform apply`
5. Require PR review for all infrastructure changes
6. Use separate state files per environment (dev/staging/prod)
