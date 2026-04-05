# Wazuh SIEM/XDR Configuration
## Project Kronos — Phase 2 Defensive Instrumentation

**Version:** Wazuh v4.12.0  
**Deployment:** Single manager, distributed agents across three cloud providers

---

## Agent Inventory

| Agent ID | Name | IP | OS | Cloud | Status |
|----------|------|----|----|-------|--------|
| 002 | Domain-control | 10.0.0.7 | Windows Server 2022 | Azure | Active |
| 004 | GCP-agent | 10.50.1.3 | Windows Server 2022 | GCP | Active |
| 005 | EC2AMAZ-UBAC2FQ | 172.16.1.186 | Windows Server 2022 | AWS | Active |
| 006 | vuln-app | 172.16.1.195 | Ubuntu 22.04.5 LTS | AWS | Active |

Screenshot: `../../docs/screenshots/phase2/fig05_wazuh_agents_multicloud.png`

---

## Detections During Simulation

### EventID 4672 — Privilege Escalation

Wazuh detected abnormal special privilege assignment on the Juice Shop VM during Stage 1 post-exploitation. 32 hits logged. SeAuditPrivilege and SeImpersonatePrivilege were assigned to a non-system process. This corresponded to the attacker establishing elevated access after initial foothold.

Screenshot: `../../docs/screenshots/phase3/fig11_wazuh_eventid4672_privilege.png`

### EventID 4625 — Failed Login Attempts

5,158 hits for EventID 4625 captured over the simulation window. Security ID S-1-0-0 (null session). This corresponds to brute-force and credential stuffing activity during the lateral movement phase.

Screenshot: `../../docs/screenshots/phase3/fig12_wazuh_eventid4625_failed_login.png`

### EventID 4698 — Scheduled Task Creation

2 hits for EventID 4698 detected on the domain controller. Account: `KRONOS\Domain-controller11`. This detected the persistence mechanism established during Stage 3 of the attack chain.

Screenshot: `../../docs/screenshots/phase3/fig13_wazuh_eventid4698_persistence.png`

---

## Recommended Correlation Rules

Based on simulation findings, the following Wazuh custom rules should be added.

```xml
<!-- Privilege escalation detection -->
<rule id="100001" level="12">
  <if_sid>18107</if_sid>
  <field name="data.win.system.eventID">4672</field>
  <description>Abnormal privilege assignment — possible escalation attempt</description>
  <group>privilege_escalation,attack</group>
</rule>

<!-- Brute force detection -->
<rule id="100002" level="10" frequency="10" timeframe="60">
  <if_matched_sid>18107</if_matched_sid>
  <field name="data.win.system.eventID">4625</field>
  <description>Multiple failed logins — possible brute force</description>
  <group>authentication_failure,brute_force</group>
</rule>

<!-- Persistence via scheduled task -->
<rule id="100003" level="12">
  <if_sid>18107</if_sid>
  <field name="data.win.system.eventID">4698</field>
  <description>Scheduled task created — possible persistence mechanism</description>
  <group>persistence,attack</group>
</rule>
```

---

## Integration Architecture

```
AWS VMs          Azure VMs        GCP VMs
(2 agents)       (1 agent)        (1 agent)
     |               |                |
     +---------------+----------------+
                     |
              Wazuh Agent (TCP 1514)
                     |
             Wazuh Manager (Central)
                     |
             Wazuh Dashboard (Kibana)
```
