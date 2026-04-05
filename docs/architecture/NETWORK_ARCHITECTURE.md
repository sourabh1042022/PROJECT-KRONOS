# Multi-Cloud Network Architecture
## Project Kronos — Phase 1 Technical Specification

---

## Design Intent

The lab environment connects three separate cloud provider networks into a single unified simulation environment. pfSense firewall appliances serve as the VPN backbone. All inter-cloud traffic routes through IPSec tunnels — no direct public internet paths exist between environments. Keycloak, deployed in Azure, handles identity federation across all three platforms.

---

## Network Topology

```
                    pfSense
                 20.194.8.140
                      |
       ---------------+---------------
       |                             |
  AWS VPC                       Azure VNet
  Tunnel CIDR:                  Tunnel CIDR:
  169.254.76.96/30              169.254.188.236/30
                                       |
                                  GCP VPC
                                  10.0.0.0/24
```

---

## IPSec Tunnel Configurations

### Tunnel 1 — pfSense to AWS

| Parameter | Value |
|-----------|-------|
| Local host | 20.194.8.140 |
| Remote host | 3.113.226.41 |
| Inside IPv4 CIDR | 169.254.76.96/30 |
| IKE version | IKEv1 |
| Role | Responder |
| Encryption | AES-CBC 128 |
| Authentication | HMAC-SHA1-96 |
| DH group | MODP 1024 |
| Status | Established |

### Tunnel 2 — pfSense to Azure

| Parameter | Value |
|-----------|-------|
| Local host | 20.194.8.140 |
| Remote host | 18.176.110.0 |
| Inside IPv4 CIDR | 169.254.188.236/30 |
| IKE version | IKEv1 |
| Role | Responder |
| Encryption | AES-CBC 128 |
| Authentication | HMAC-SHA1-96 |
| DH group | MODP 1024 |
| Status | Established |

### Tunnel 3 — pfSense to GCP

| Parameter | Value |
|-----------|-------|
| Local host | 20.194.8.140 |
| Remote host | 34.59.82.239 |
| IKE version | IKEv2 |
| Role | Initiator |
| Encryption | AES-CBC 256 |
| Authentication | HMAC-SHA2-256-128 |
| DH group | MODP 2048 |
| Rekey timer | 14550s |
| Status | Established |

---

## Host Inventory

| Cloud | Resource | IP / CIDR | Purpose |
|-------|----------|-----------|---------|
| pfSense | Firewall appliance | 20.194.8.140 | Central VPN gateway |
| AWS | VPC | 10.0.0.0/16 | AWS workloads |
| AWS | EC2 vuln-app (Ubuntu) | 172.16.1.195 | OWASP Juice Shop target |
| AWS | EC2 domain-ctrl (Windows) | 172.16.1.186 | Windows domain endpoint |
| AWS | S3 bucket | cloud-lab-states | Terraform state (misconfigured) |
| Azure | VNet | 172.16.0.0/16 | Azure workloads |
| Azure | Domain Controller (Windows) | 10.0.0.7 | Azure Windows endpoint |
| GCP | VPC | 10.0.0.0/24 | GCP workloads |
| GCP | GCP-agent (Windows) | 10.50.1.3 | GCP Windows endpoint |
| GCP | Cloud Storage | crownjewel-lab | Crown jewel dataset |

---

## Keycloak Identity Federation

Keycloak is deployed in Azure and serves as the central IdP for all three cloud environments. All authentication flows route through Keycloak's SSO realm.

- Realm: master
- Federation: AWS IAM, Azure AD, GCP IAM
- MFA enforced at the realm level for all users
- Service accounts subject to the same governance as human accounts

Screenshot: `../screenshots/phase1/fig02_keycloak_dashboard.png`

---

## Evidence of Deployment

All four screenshots below were captured from live infrastructure during Phase 1 of the simulation.

`../screenshots/phase1/fig01_pfsense_ipsec_overview.png`
pfSense IPSec status page showing all three tunnels (con1, con2, con3) in Established state.

`../screenshots/phase1/fig02_keycloak_dashboard.png`
Keycloak master realm live in Azure with SSO and identity federation configured.

`../screenshots/phase1/fig03_gcp_vpn_tunnel_status.png`
GCP VPN tunnel detail: gcp-to-pfsense-tunnel1, Status: Tunnel is up and running. Remote peer: 20.194.8.140.

`../screenshots/phase1/fig04_aws_vpn_tunnel_status.png`
AWS VPN connection showing Tunnel 1 (3.113.226.41) and Tunnel 2 (18.176.110.0), both UP and Available.
