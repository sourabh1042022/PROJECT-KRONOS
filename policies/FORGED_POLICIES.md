# Forged Zero-Trust Policies
## Project Kronos — Phase 4 Policy Forging

25 policies derived directly from live adversarial simulation evidence. Each policy documents the observed gap, the evidence that exposed it, and the enforceable control derived from it.

---

## Domain 1 — Identity and Authentication

### Policy 1 — Multi-Factor Authentication Enforcement

**Observed gap:** Compromised static credentials were used across all four stages of the attack chain without any MFA challenge. Once extracted from the Terraform state file, the credentials worked immediately with no additional verification.

**Forged policy:** MFA is mandatory for all accounts including service accounts. Any authentication attempt without MFA is blocked at the IdP level via Keycloak. Compliance is validated through Wazuh SIEM alerts and monthly IAM audits. No exceptions are permitted for programmatic accounts.

---

### Policy 2 — Privileged Access Management

**Observed gap:** Standing admin privileges on the Juice Shop VM enabled rapid privilege escalation after initial foothold. Wazuh detected EventID 4672 (abnormal privilege assignment) but the escalation had already completed by that point.

**Evidence:** `docs/screenshots/phase3/fig11_wazuh_eventid4672_privilege.png`

**Forged policy:** All privileged access requires Just-In-Time approval via a PAM system with automatic session expiration after one hour. Standing admin rights are prohibited. All privileged sessions are logged to Wazuh SIEM.

---

### Policy 3 — Service Account Credential Management

**Observed gap:** AWS access keys and GCP service account keys were long-lived static credentials stored in plaintext in a Terraform state file in S3. Once the bucket was accessed, the attacker had persistent multi-cloud access with no time constraint.

**Evidence:** `docs/screenshots/phase2/fig09_aws_s3_azure_keys.png`

**Forged policy:** All service accounts must use short-lived tokens with a maximum one-hour lifetime. Long-lived static keys are prohibited. Key rotation is enforced automatically via IAM policies and Cloud Custodian. Source repositories and Terraform state files are scanned weekly for exposed credentials.

---

### Policy 4 — Device-Based Access Control

**Observed gap:** The attack was executed from a Kali Linux machine — an unmanaged, non-compliant endpoint. No conditional access controls prevented authentication from an unrecognized device.

**Forged policy:** Conditional access policies deny authentication from non-compliant endpoints. Device compliance is verified via EDR agent health checks and OS patch level. Enforcement is applied via Azure Conditional Access and equivalent controls on AWS and GCP.

---

## Domain 2 — Device and Endpoint Security

### Policy 5 — Endpoint Detection and Response

**Observed gap:** Several attack stages moved between VMs with no host-based detection. Gaps in Wazuh agent coverage meant certain VMs generated no telemetry during the attack.

**Evidence:** `docs/screenshots/phase2/fig05_wazuh_agents_multicloud.png`

**Forged policy:** All endpoints must run Wazuh EDR agents reporting in real time to the centralized SIEM. Devices that fail health checks or lose agent connectivity are automatically blocked from network access until compliance is restored.

---

### Policy 6 — Local Administrator Account Elimination

**Observed gap:** Standing local admin accounts on servers enabled rapid privilege escalation. EventID 4672 confirmed this pattern during the simulation.

**Forged policy:** All standing local admin privileges are removed across all VMs and endpoints. Admin elevation is granted only via JIT PAM, time-limited to the approved task. Wazuh monitors all elevation events (EventID 4672/4104) continuously.

---

### Policy 7 — Disk Encryption

**Observed gap:** Data exfiltration from GCP Cloud Storage showed that once access was obtained, data was immediately readable with no additional barriers.

**Forged policy:** All cloud VM disks use platform-managed or customer-managed encryption keys. Non-compliant instances are flagged by Cloud Custodian. Encryption compliance is audited monthly.

---

## Domain 3 — Network and Segmentation

### Policy 8 — Micro-Segmentation

**Observed gap:** Once inside AWS, the attacker moved freely toward Azure and GCP. Overly permissive security groups and network ACLs allowed east-west traffic between cloud environments without restriction.

**Forged policy:** Micro-segmentation is enforced using AWS Security Groups, Azure NSGs, and GCP VPC Firewall Rules. Only explicitly whitelisted traffic flows are permitted between workloads. All inter-VM communication rules are audited monthly.

---

### Policy 9 — Inter-Cloud Traffic Encryption

**Observed gap:** Some inter-cloud communications were transmitted without consistent end-to-end encryption, creating interception opportunities during transit.

**Evidence:** `docs/screenshots/phase1/fig01_pfsense_ipsec_overview.png`

**Forged policy:** All inter-cloud traffic routes through pfSense IPSec VPN tunnels using AES-256-GCM with SHA-512 authentication. All non-tunneled inter-cloud traffic is blocked at the pfSense firewall level. Compliance is validated monthly.

---

### Policy 10 — Egress Traffic Control

**Observed gap:** Crown jewel data was exfiltrated from GCP Cloud Storage without triggering any egress controls. Outbound traffic was not filtered or monitored.

**Forged policy:** Default-deny on all outbound traffic. Only whitelisted destination IPs and ports are permitted. Every blocked egress attempt is logged and alerted in Wazuh SIEM. Exfiltration detection rules are configured based on volume and destination anomalies.

---

## Domain 4 — Applications and Workloads

### Policy 11 — Identity-Based Application Security

**Observed gap:** Terraform state files stored in S3 contained plaintext AWS and Azure credentials. The application had no mechanism to rotate or protect those credentials dynamically.

**Evidence:** `docs/screenshots/phase2/fig09_aws_s3_azure_keys.png`

**Forged policy:** All workloads authenticate using IAM roles or managed identities. Hardcoded credentials in code, state files, or environment variables are prohibited. Repositories and Terraform state backends are scanned for secrets weekly.

---

### Policy 12 — Web Application Security

**Observed gap:** OWASP Juice Shop was exploitable via SQL injection and XSS, enabling the attacker to establish initial foothold and extract credentials. These are well-understood, preventable vulnerability classes.

**Forged policy:** All production web applications deploy either a RASP agent or WAF rules blocking SQLi and XSS patterns. All blocked attempts are logged to Wazuh. Security testing is conducted quarterly against production applications.

---

### Policy 13 — API Security

**Observed gap:** Certain cloud management APIs were accessible without strong authentication, enabling resource enumeration and foothold expansion.

**Forged policy:** All APIs are fronted by an API gateway enforcing authentication and authorization. Rate limiting and comprehensive logging are enabled on all API endpoints. API configurations are validated monthly.

---

## Domain 5 — Data and Storage

### Policy 14 — Data Encryption at Rest

**Observed gap:** Crown jewel data in GCP Cloud Storage was immediately readable once a valid service account key was obtained. No additional encryption layer provided a secondary barrier.

**Evidence:** `docs/screenshots/phase2/fig10_gcp_crownjewel_bucket.png`

**Forged policy:** All cloud storage buckets and databases enforce AES-256 encryption at rest using customer-managed keys. Unencrypted resources are flagged and blocked by Cloud Custodian. Encryption compliance is audited monthly.

---

### Policy 15 — Least-Privilege Data Access

**Observed gap:** Over-provisioned IAM permissions allowed the attacker to access resources well beyond the scope of the initial compromise. A single extracted key provided access to the entire crown jewel dataset.

**Forged policy:** Least-privilege IAM roles are enforced across all cloud platforms. No role may grant broader access than the minimum required for the specific function. All data access is logged in Wazuh and reviewed monthly.

---

### Policy 16 — Data Loss Prevention

**Observed gap:** GCP Cloud Storage access using a stolen service account key was not detected in real time. The exfiltration of crown jewel data completed before any alert fired.

**Forged policy:** DLP rules cover all cloud storage services. Anomalous access patterns trigger alerts forwarded to Wazuh. Policy violations trigger automatic network ACL-based containment.

---

### Policy 17 — Log Data Protection

**Observed gap:** Terraform execution logs captured during the simulation contained plaintext credential fragments visible in log archives.

**Forged policy:** All logs are tokenized and masked before SIEM ingestion. Sensitive field patterns including credentials, PII, and keys are automatically redacted at the log forwarder level. Compliance is validated monthly.

---

### Policy 18 — Cloud Storage Access Control

**Observed gap:** A long-lived service account key provided persistent access to the GCP crown jewel bucket with no expiry and no access monitoring in place.

**Forged policy:** All cloud storage operations require signed URLs or scoped access tokens. Access keys are rotated every seven days. All bucket access events are forwarded to Wazuh.

---

## Domain 6 — Monitoring and Governance

### Policy 19 — Centralized Log Collection

**Observed gap:** During the simulation, attack actions in Azure and GCP were not immediately visible in the centralized Wazuh instance due to logging gaps and misconfigured log forwarders.

**Forged policy:** All cloud platform logs, VM logs, application logs, and network logs are forwarded to the centralized Wazuh instance. Log ingestion health is validated daily. Any ingestion gap triggers an immediate alert.

---

### Policy 20 — Automated Threat Detection

**Observed gap:** Lateral movement between cloud environments had an extended dwell time before detection. The full attack chain from initial access to exfiltration took under one hour and was largely undetected in real time.

**Evidence:** `docs/screenshots/phase3/fig12_wazuh_eventid4625_failed_login.png`

**Forged policy:** Wazuh correlation rules detect abnormal login patterns (EventID 4625), privilege escalation (EventID 4672), persistence mechanisms (EventID 4698), and cross-cloud authentication anomalies. All high-severity alerts are triaged within five minutes of generation.

---

### Policy 21 — Automated Response

**Observed gap:** Response to the cross-cloud attack pivots was delayed because containment required manual intervention. By the time manual containment was executed, the attack had already moved to the next stage.

**Forged policy:** Automated network quarantine scripts integrated with Wazuh active response isolate compromised VMs and endpoints upon confirmed threat detection. Automated response playbooks are reviewed and updated quarterly.

---

### Policy 22 — Cloud Custodian Policy Maintenance

**Observed gap:** Several Cloud Custodian governance rules failed to fire timely alerts during the simulation due to stale configurations that had not been updated to reflect the current environment.

**Forged policy:** All Cloud Custodian policies are audited monthly. Rules are updated within 72 hours of discovering new cloud misconfigurations or simulation findings. All rule changes are version-controlled via Git and require peer review.

---

### Policy 23 — Change Management

**Observed gap:** A Terraform state file was exposed in a world-readable S3 bucket due to an unreviewed infrastructure change. The change introduced the vulnerability that enabled Stage 2 of the attack chain.

**Forged policy:** All infrastructure changes require PR-based approval with at least one peer review. Automated pre-commit hooks enforce secret scanning before any code is committed. Terraform state backends must enforce encryption and access logging.

---

### Policy 24 — Continuous Security Improvement

**Observed gap:** Several vulnerabilities had existed in the environment for an extended period without detection because no regular adversarial testing was in place.

**Forged policy:** Quarterly adversarial simulation exercises are conducted. After each exercise, policies, detection rules, and monitoring configurations are updated based on findings. Mean-time-to-detect and mean-time-to-respond are tracked as core KPIs.

---

### Policy 25 — Zero-Trust Continuous Verification

**Observed gap:** The full four-stage attack chain succeeded end-to-end. Individual Zero-Trust controls were present but were not functioning as a cohesive, continuously verified framework. Lateral movement and exfiltration exposed segmentation failures at every transition point.

**Forged policy:** Zero-Trust is continuously enforced via Wazuh real-time monitoring, IAM least-privilege enforcement, pfSense-encrypted inter-cloud tunnels, and monthly micro-segmentation audits. Non-compliance triggers automated alerts and temporary network containment of the affected resource.

---

## Summary

| Domain | Policies | Core Finding |
|--------|----------|-------------|
| Identity and Authentication | 1–4 | Static credentials and absent MFA enabled full multi-cloud compromise |
| Device and Endpoint | 5–7 | Agent coverage gaps left multiple attack stages undetected |
| Network and Segmentation | 8–10 | Flat network allowed unrestricted east-west movement between clouds |
| Applications and Workloads | 11–13 | Hardcoded credentials in Terraform state were the single point of failure |
| Data and Storage | 14–18 | No egress monitoring, no DLP, and no key rotation on storage access |
| Monitoring and Governance | 19–25 | Detection coverage existed but was insufficient to catch the full chain |
