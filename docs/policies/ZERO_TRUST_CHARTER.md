# Foundational Multi-Cloud Zero-Trust Charter
## Project Kronos — Phase 1 Deliverable

**Version:** 1.0  
**Date:** August 15, 2025  
**Status:** Active — validated through live adversarial simulation

---

## Purpose

This charter defines the foundational Zero-Trust security principles governing the Project Kronos multi-cloud environment. Zero-Trust operates on the premise that no entity — internal or external — is trusted by default. Every access request must be verified, every session must be authorized, and every action must be logged and auditable.

This charter was written before the simulation began. The live exercise exposed specific gaps in each domain. Those gaps are documented in the forged policies at `../../policies/FORGED_POLICIES.md`.

---

## Domain 1 — Identity

**Authentication and MFA**
All access to cloud resources requires strong authentication. Multi-factor authentication is mandatory for all human accounts and all service accounts using interactive flows. Static API keys are treated as legacy credentials requiring immediate migration to dynamic token-based identity.

**Authorization and Least Privilege**
Authorization follows the principle of least privilege. Every identity is granted the minimum set of permissions required for its function. Permissions are reviewed quarterly and revoked when no longer needed.

**Identity Federation**
Keycloak serves as the central Identity Provider federating identity across AWS, Azure, and GCP. All authentication flows route through Keycloak's SSO framework, providing a single point of identity governance across the multi-cloud estate.

**Service Account Credential Lifecycle**
Service accounts are treated as privileged identities and are subject to the same MFA, least-privilege, and audit requirements as human accounts. Credentials have a maximum lifetime of one hour and are rotated automatically.

---

## Domain 2 — Network

**Inter-Cloud Connectivity**
All connectivity between AWS VPC, Azure VNet, and GCP VPC must traverse authenticated, encrypted channels. Direct public internet paths between cloud environments are prohibited.

**Traffic Segmentation**
East-west traffic between workloads is denied by default. Explicit allowlist rules govern every permitted communication path. Micro-segmentation is implemented at the workload level using cloud-native security controls.

**Encryption in Transit**
All inter-cloud traffic is encrypted using IPSec tunnels with AES-256-GCM and SHA-512 authentication, managed via pfSense. All application-layer traffic uses TLS 1.2 minimum, TLS 1.3 preferred.

**Network Monitoring**
All network flows are logged. Anomalous traffic patterns trigger automated alerts in Wazuh SIEM. Egress traffic is filtered and monitored via pfSense firewall rules.

---

## Domain 3 — Endpoint

**Security Baseline**
All VMs across AWS, Azure, and GCP must meet a minimum security baseline before joining the environment. This includes OS patching within 72 hours of critical updates, disabled unnecessary services, and Wazuh agent installation.

**Endpoint Detection and Response**
All compute endpoints must run Wazuh EDR agents with real-time telemetry reporting to the centralized SIEM. Endpoints that lose agent connectivity are automatically quarantined.

**Device Compliance**
Devices must pass automated health checks covering patch level, EDR agent status, and disk encryption compliance. Non-compliant devices are denied access until remediation is verified.

**Privileged Access**
No standing local administrator accounts are permitted. All privileged access is granted via Just-In-Time PAM with automatic session expiry. All privileged actions are logged and auditable.

---

## Domain 4 — Data

**Data Classification**
All data stored in the environment is classified into one of three tiers: Public, Internal, or Restricted. Crown jewel data is classified as Restricted and subject to the most stringent access and encryption controls.

**Encryption at Rest and in Transit**
All Restricted and Internal data is encrypted at rest using AES-256 with customer-managed keys. All data in transit uses TLS 1.3 or IPSec. No unencrypted storage of Restricted data is permitted under any circumstances.

**Access Control for Sensitive Data**
Access to Restricted data requires valid identity via Keycloak, MFA challenge, explicit IAM role assignment, and logging of all access events. All access is reviewed monthly.

**Data Loss Prevention**
DLP rules monitor all egress paths for sensitive data patterns. Violations trigger automated containment via network ACLs and immediate SIEM alerts. Egress volume anomalies are monitored using behavioral baselines.

---

## Charter Review Process

This charter is reviewed after each adversarial simulation exercise, quarterly as part of the security governance cycle, and immediately following any confirmed security incident. Updates are version-controlled in this repository.
