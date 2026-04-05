# Project Kronos
## Multi-Cloud Zero-Trust Simulation and Policy Forging

**Author:** Sourabh Kumar  
**Role:** Security Analyst  
**Approval Authority:** Director of Security  
**Date:** August 15, 2025  
**Budget:** Zero — built entirely on free-tier cloud accounts and open-source tooling

---

## What This Project Is

Project Kronos is a live adversarial simulation designed to stress-test zero-trust security principles across three cloud environments simultaneously. Rather than deriving security policy from frameworks or theoretical models, the project runs real attack chains against real infrastructure and builds policy from what actually happens.

The environment spans AWS, Microsoft Azure, and Google Cloud Platform, interconnected through a pfSense mesh VPN with Keycloak handling centralized identity. Wazuh provides cross-cloud SIEM coverage, and Cloud Custodian enforces automated governance policies on all three platforms.

The attack chain moves in four stages: initial access through a vulnerable web application in AWS, lateral movement into Azure by harvesting credentials from a misconfigured S3 bucket, pivoting to GCP through Azure VM metadata exploitation, and finally exfiltrating crown jewel data from GCP Cloud Storage. Every stage was executed, logged, and analyzed. The 25 policies in this repository were written in direct response to what was observed.

---

## Repository Layout

```
project-kronos/
│
├── README.md
├── ROLES_AND_RESPONSIBILITIES.md
│
├── docs/
│   ├── architecture/
│   │   └── NETWORK_ARCHITECTURE.md
│   ├── policies/
│   │   └── ZERO_TRUST_CHARTER.md
│   └── screenshots/
│       ├── README.md
│       ├── phase1/                    <- Infrastructure deployment
│       ├── phase2/                    <- Monitoring and attack preparation
│       └── phase3/                    <- Live attack and detection evidence
│
├── infrastructure/
│   └── terraform/
│       └── README.md
│
├── monitoring/
│   ├── wazuh/
│   │   └── README.md
│   └── cloud-custodian/
│       └── policies/
│           └── governance-policies.yml
│
├── red-team/
│   └── ATTACK_CHAIN.md
│
└── policies/
    └── FORGED_POLICIES.md
```

---

## Environment Overview

Three cloud providers are connected via pfSense IPSec tunnels running AES-256-GCM with SHA-512 authentication. All inter-cloud traffic routes through the VPN mesh — no direct public internet paths between environments.

Keycloak is deployed in Azure and acts as the central Identity Provider, federating identity across AWS IAM, Azure AD, and GCP IAM through a single SSO realm.

Wazuh agents run on every VM across all three clouds and report to a centralized manager. Four agents were active during the simulation: two in AWS (one Windows, one Ubuntu), one in Azure (Windows), and one in GCP (Windows).

Cloud Custodian is deployed across all three platforms and enforces five governance policies covering EC2 tag enforcement, Azure public IP cleanup, GCP SSH exposure monitoring, cross-cloud storage access control, and cross-cloud tagging standards.

```
                    pfSense (20.194.8.140)
                          |
          ----------------+----------------
          |                               |
     AWS VPC                         Azure VNet
  169.254.76.96/30               169.254.188.236/30
  - EC2 (vuln-app)               - Domain Controller
  - EC2 (domain-ctrl)            - Azure VMs
  - S3 (cloud-lab-states)        - Keycloak IdP
          |                               |
          +---------------+---------------+
                          |
                       GCP VPC
                      10.0.0.0/24
                   - GCP agent VM
                   - Cloud Storage (crownjewel-lab)
```

---

## Attack Chain Summary

The simulation executed a four-stage cross-cloud attack.

**Stage 1 — AWS Initial Compromise**
Exploited OWASP Juice Shop running in AWS EC2 via SQL injection and XSS. Extracted AWS access keys from application environment variables and established initial foothold.

**Stage 2 — Azure Lateral Movement**
Used compromised AWS credentials to access S3 bucket `cloud-lab-states`. Found `azure_sp.json` — a Terraform state file containing plaintext Azure service principal credentials. Authenticated to Azure using those credentials and enumerated the Azure environment.

**Stage 3 — GCP Service Account Compromise**
Queried the Azure VM metadata service (IMDS) from the compromised Azure VM. Extracted a GCP service account key stored on the VM. Authenticated to GCP and identified the `crownjewel-lab` Cloud Storage bucket.

**Stage 4 — Crown Jewel Exfiltration**
Accessed `crownjewel-lab` using the extracted service account key. Downloaded `crownjewel.txt` — the designated sensitive dataset. Attack chain complete.

---

## What Wazuh Detected

Three attack-related events were captured during the simulation.

EventID 4672 — abnormal privilege assignment on the Juice Shop VM — detected during Stage 1 post-exploitation. 32 hits logged.

EventID 4625 — failed login attempts — 5,158 hits captured over the simulation window, corresponding to credential stuffing and brute-force activity during lateral movement.

EventID 4698 — scheduled task creation on the domain controller — 2 hits, corresponding to the persistence mechanism established during the attack. Account: `KRONOS\Domain-controller11`.

The attack chain from initial access through crown jewel exfiltration was not fully detected in real time. Stages 2, 3, and 4 generated no alerts. This gap is the primary driver of the forged policies.

---

## Cloud Custodian Governance Policies

Five policies were deployed and validated across all three cloud platforms.

1. **AWS EC2 Tag Enforcement** — stops EC2 instances without mandatory Owner tags and marks them non-compliant
2. **Azure Public IP Cleanup** — automatically deletes unused Azure public IPs not attached to any resource
3. **GCP SSH Security Monitoring** — detects GCP firewall rules allowing SSH from 0.0.0.0/0
4. **Multi-Cloud Storage Security** — blocks public access on AWS S3, Azure Blob storage, and GCP Cloud Storage buckets
5. **Cross-Cloud Tagging Enforcement** — enforces Owner tags across AWS EC2, Azure VMs, and GCP instances with automated remediation

Policy YAML: `monitoring/cloud-custodian/policies/governance-policies.yml`

---

## Forged Policies

25 policies were written directly from simulation evidence, covering six domains.

| Domain | Count |
|--------|-------|
| Identity and Authentication | 4 |
| Device and Endpoint Security | 3 |
| Network and Segmentation | 3 |
| Applications and Workloads | 3 |
| Data and Storage | 5 |
| Monitoring and Governance | 7 |

Each policy documents the observed gap, the attack evidence that exposed it, and the enforceable control derived from it.

Full policy set: `policies/FORGED_POLICIES.md`

---

## Proof of Concept Screenshots

All screenshots from the live simulation are in `docs/screenshots/`, organized by phase.

Phase 1 covers infrastructure deployment: pfSense IPSec tunnels established, Keycloak IdP live, GCP and AWS VPN tunnel status confirmed active.

Phase 2 covers monitoring and attack preparation: Wazuh agents across all three clouds, Cloud Custodian v0.9.46 deployed, OWASP Juice Shop running, misconfigured S3 bucket with seeded credentials, GCP crown jewel bucket.

Phase 3 covers live detection: EventID 4672 privilege escalation, EventID 4625 brute-force activity (5,158 hits), EventID 4698 scheduled task persistence.

---

## Technology Stack

| Category | Tool |
|----------|------|
| Cloud platforms | AWS, Azure, GCP (all free tier) |
| Network security | pfSense (IPSec mesh, AES-256-GCM + SHA-512) |
| Identity management | Keycloak (central IdP, SSO, federation) |
| SIEM / XDR | Wazuh (agents on all VMs, correlation rules) |
| Cloud posture management | Cloud Custodian v0.9.46 |
| Infrastructure as Code | Terraform |
| Attack platform | Kali Linux, Metasploit Framework, Legion |
| Vulnerable target | OWASP Juice Shop |

---

## Disclaimer

This project was conducted in a controlled, isolated lab environment using deliberately vulnerable infrastructure provisioned for simulation purposes. All attack techniques were executed with explicit authorization within a sandboxed environment. Nothing in this repository should be used against systems you do not own or have written permission to test.
