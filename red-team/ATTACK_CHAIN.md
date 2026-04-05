# Attack Chain Documentation
## Project Kronos — Phase 3 Adversarial Simulation

All techniques described in this document were executed in a controlled, isolated lab environment with explicit authorization. This documentation exists for defensive research and policy derivation purposes.

---

## Objectives

1. Demonstrate that a single public-facing vulnerability can cascade into full multi-cloud compromise.
2. Generate concrete evidence for policy development by executing real attack techniques against real infrastructure.
3. Measure detection coverage and identify gaps in the existing defensive stack.

---

## Attack Infrastructure

| Component | Tool |
|-----------|------|
| Primary attack platform | Kali Linux |
| Exploitation framework | Metasploit Framework |
| Network discovery | Legion |
| Reconnaissance | OSINT frameworks |
| Credential validation | AWS CLI, Azure CLI, gcloud |

---

## Stage 1 — AWS Initial Compromise

**Target:** OWASP Juice Shop running on EC2 (172.16.1.195)

SQL injection was used to extract user data and session tokens from the Juice Shop application. Cross-site scripting was leveraged for session hijacking. API endpoint enumeration identified admin routes and application environment variables containing AWS access keys.

AWS credentials were validated using `aws sts get-caller-identity`. Initial foothold established.

**Evidence:** `../docs/screenshots/phase2/fig08_owasp_juiceshop_dashboard.png`

---

## Stage 2 — Azure Lateral Movement

**Target:** S3 bucket `cloud-lab-states`

Using the compromised AWS credentials, S3 buckets were enumerated. The bucket `cloud-lab-states` was accessible and contained `azure_sp.json` — a Terraform state file storing Azure service principal credentials in plaintext.

The Terraform state file was downloaded and parsed. Azure service principal ID and secret were extracted. Authentication to Azure succeeded using those credentials. Azure resources, VMs, and service configurations were enumerated.

The single misconfigured S3 bucket — world-readable, containing a Terraform state file — was the pivot point for the entire cross-cloud attack.

**Evidence:** `../docs/screenshots/phase2/fig09_aws_s3_azure_keys.png`

---

## Stage 3 — GCP Service Account Compromise

**Target:** Azure VM metadata service on the compromised Azure VM

The Azure Instance Metadata Service was queried from the compromised Azure VM at `http://169.254.169.254/metadata/instance`. A GCP service account key file stored on the Azure VM was located and extracted. Authentication to GCP was established using `gcloud auth activate-service-account`. The `crownjewel-lab` Cloud Storage bucket was identified.

---

## Stage 4 — Crown Jewel Exfiltration

**Target:** GCP Cloud Storage bucket `crownjewel-lab`

Using the extracted service account key, the `crownjewel-lab` bucket was accessed. `crownjewel.txt` was identified and downloaded. The attack chain was complete — from a public web application vulnerability to sensitive data exfiltration across three cloud providers.

**Evidence:** `../docs/screenshots/phase2/fig10_gcp_crownjewel_bucket.png`

---

## Detection Coverage

| Stage | Action | Detected | Method |
|-------|--------|----------|--------|
| 1 | SQLi on Juice Shop | No | — |
| 1 | AWS key extraction | No | — |
| 2 | S3 bucket access | No | — |
| 2 | Terraform state download | No | — |
| 2/3 | Privilege escalation | Yes | Wazuh EventID 4672 |
| 3 | Azure IMDS query | No | — |
| 3 | GCP key extraction | No | — |
| 3/4 | Brute-force activity | Yes | Wazuh EventID 4625 (5,158 hits) |
| 3 | Scheduled task persistence | Yes | Wazuh EventID 4698 |
| 4 | GCP bucket access | No | — |
| 4 | Crown jewel exfiltration | No | — |

The blue team detected post-exploitation persistence and privilege escalation activity but missed the full attack chain. Initial access, lateral movement, credential harvesting, and data exfiltration all completed without generating alerts. This detection gap is the primary evidence base for the forged policies.

---

## Dwell Time by Stage

| Stage | Time from start | Detection |
|-------|----------------|-----------|
| Stage 1 — AWS initial access | T+0 | Not detected |
| Stage 2 — Azure pivot | T+15 min | Not detected |
| Stage 3 — GCP compromise | T+35 min | Partial (post-escalation only) |
| Stage 4 — Exfiltration | T+55 min | Not detected |

---

## Root Cause Analysis

The attack succeeded end-to-end for three primary reasons.

First, a single misconfigured S3 bucket containing a Terraform state file with plaintext credentials enabled lateral movement from AWS into Azure. One misconfiguration unlocked access to a second cloud provider entirely.

Second, all service account credentials used during the attack were long-lived static keys with no expiry. Once harvested, they remained valid for the entire duration of the exercise.

Third, egress monitoring was insufficient. Data exfiltration from GCP Cloud Storage to an external destination generated no alerts. There were no DLP controls, no egress filtering, and no anomaly detection on storage access patterns.
