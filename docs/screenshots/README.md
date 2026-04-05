# Screenshots — Proof of Concept Evidence
## Project Kronos Live Simulation

All screenshots in this directory were captured during the live Project Kronos simulation. They are organized by phase and serve as the evidence base for the forged policies.

---

## Phase 1 — Infrastructure Deployment

**fig01_pfsense_ipsec_overview.png**
pfSense IPSec status page. Three tunnels (con1, con2, con3) connecting AWS, Azure, and GCP are all in Established state. Encryption: AES-CBC. Authentication: HMAC-SHA1-96 (AWS, Azure) and HMAC-SHA2-256-128 (GCP).

**fig02_keycloak_dashboard.png**
Keycloak master realm deployed in Azure. SSO and identity federation configured. Welcome screen confirms the realm is live and accepting connections.

**fig03_gcp_vpn_tunnel_status.png**
GCP Network Connectivity view. Tunnel `gcp-to-pfsense-tunnel1`: Status — Tunnel is up and running. Remote peer: 20.194.8.140 (pfSense). VPC network: kronos-gcp-vpc. Region: us-central1.

**fig04_aws_vpn_tunnel_status.png**
AWS VPN Connections view. Connection vpn-099a08a3418bc2e18. Tunnel 1 (3.113.226.41) and Tunnel 2 (18.176.110.0) both showing Status: Up, Provisioning: Available.

---

## Phase 2 — Monitoring and Attack Preparation

**fig05_wazuh_agents_multicloud.png**
Wazuh Endpoints dashboard. 4 active agents: Domain-control (Azure), GCP-agent (GCP), EC2AMAZ-UBAC2FQ (AWS), vuln-app (AWS Ubuntu). All running Wazuh v4.12.0, all Active.

**fig06_cloud_custodian_install.png**
pip install output showing c7n_azure, c7n_gcp, c7n_mailer, and all dependencies installing successfully on the Cloud Custodian management host.

**fig07_custodian_version_check.png**
Terminal output: `custodian version 0.9.46`. Confirms Cloud Custodian is deployed and operational.

**fig08_owasp_juiceshop_dashboard.png**
OWASP Juice Shop running in AWS EC2. The storefront is live with product listings visible. This is the attack target used in Stage 1 of the attack chain.

**fig09_aws_s3_azure_keys.png**
AWS S3 bucket `cloud-lab-states` containing `azure_sp.json` (695 bytes, Standard storage class, created August 18 2025). This Terraform state file contained plaintext Azure service principal credentials and was the pivot point for Stage 2.

**fig10_gcp_crownjewel_bucket.png**
GCP Cloud Storage bucket `crownjewel-lab`. Contains `crownjewel.txt` (38 bytes, text/plain, Not public, created August 18 2025). This is the crown jewel data exfiltrated in Stage 4.

---

## Phase 3 — Live Attack and Detection

**fig11_wazuh_eventid4672_privilege.png**
Wazuh Discover view filtered on EventID 4672. 32 hits. Events show SeAuditPrivilege and SeImpersonatePrivilege assignment. Agent: Domain-control (SOC-Analyst role). Captured during Stage 1 post-exploitation.

**fig12_wazuh_eventid4625_failed_login.png**
Wazuh Discover view filtered on EventID 4625. 5,158 hits. Message: "An account failed to log on." Security ID: S-1-0-0 (null session). Captured during lateral movement phase.

**fig13_wazuh_eventid4698_persistence.png**
Wazuh Discover view filtered on EventID 4698. 2 hits. Message: "A scheduled task was created." Account: `KRONOS\Domain-controller11`. Captured during Stage 3 — attacker establishing persistence on the domain controller.
