# Project Kronos — Roles and Responsibilities

**Project:** Multi-Cloud Zero-Trust Simulation and Policy Forging  
**Author:** Sourabh Kumar — Security Analyst  
**Approval Authority:** Director of Security  
**Date:** August 15, 2025

---

## Overview

Project Kronos was designed and executed as a solo security research engagement. All phases of the project — infrastructure design, defensive instrumentation, adversarial simulation, detection analysis, and policy forging — were carried out by one analyst across a five-week period.

---

## Responsibilities

**Cloud and Security Architecture**

Designed and built the full multi-cloud network topology spanning AWS VPC, Azure VNet, and GCP VPC. Established free-tier accounts across all three cloud providers. Deployed pfSense firewall appliances and configured site-to-site IPSec VPN tunnels between all three environments using AES-256-GCM encryption and SHA-512 authentication. Deployed and configured Keycloak as the central Identity Provider, federating identity across all three cloud platforms. Managed VM provisioning and baseline security hardening across all environments.

**Defensive Instrumentation**

Deployed Wazuh SIEM/XDR manager and installed agents on all VMs across AWS, Azure, and GCP. Configured custom correlation rules and alerting for cross-cloud threat detection. Deployed Cloud Custodian v0.9.46 across all three platforms and wrote five foundational governance policies covering tagging enforcement, public IP cleanup, SSH exposure monitoring, storage access control, and compliance automation. Built real-time monitoring dashboards and maintained centralized logging infrastructure throughout the simulation.

**Adversarial Simulation**

Established Kali Linux attack infrastructure. Conducted OSINT reconnaissance of the target environment. Developed a multi-stage attack plan and executed all four stages of the cross-cloud attack chain: initial access via OWASP Juice Shop exploitation in AWS, lateral movement through credential harvesting from a misconfigured S3 Terraform state file, pivot to GCP via Azure VM metadata service exploitation, and crown jewel data exfiltration from GCP Cloud Storage. Documented all attack methodologies, exploitation techniques, and post-exploitation findings.

**Incident Response and Detection Analysis**

Monitored Wazuh SIEM throughout the simulation. Triaged and investigated all alerts triggered during the attack chain. Analyzed detection gaps — specifically the stages of the attack that generated no alerts — and documented root causes for each gap.

**Policy Forging**

Led the post-incident review. Analyzed the full attack timeline against existing controls. Identified 25 specific security gaps observed during the simulation and wrote an enforceable policy for each one. Developed the Foundational Multi-Cloud Zero-Trust Charter. Prepared all documentation for executive review.

---

## Tooling Used

AWS (free tier), Microsoft Azure (free tier), GCP (free tier), pfSense, Keycloak, Terraform, Wazuh SIEM/XDR, Cloud Custodian v0.9.46, Kali Linux, Metasploit Framework, Legion, OWASP Juice Shop

---

## Deliverables

- Full multi-cloud lab environment with pfSense VPN mesh across AWS, Azure, and GCP
- Five Cloud Custodian governance policies deployed and validated across all three platforms
- Twenty-five Zero-Trust policies derived directly from live adversarial simulation evidence
- Thirteen proof-of-concept screenshots documenting the full simulation lifecycle
- Foundational Multi-Cloud Zero-Trust Charter
- Complete attack chain documentation with post-exploitation analysis
