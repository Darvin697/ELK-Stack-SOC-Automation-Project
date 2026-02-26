# Phase 4 — Incident Response & Ticketing (Days 23–30)

## Overview

The final phase tied everything together by integrating osTicket with the Elastic SIEM to automate the creation of incident tickets from security alerts. Full adversary simulations were conducted, and the complete **Alert → Ticket → Investigation → Resolution** lifecycle was documented — simulating a real SOC analyst workflow.

---

## 4.1 osTicket Installation & Configuration

osTicket is an open-source ticketing system used to manage and track security incidents. It was deployed on a dedicated Ubuntu VM.

### Installation Steps:
1. Installed LAMP stack (Apache, MySQL, PHP) on the osTicket Ubuntu VM
2. Downloaded and extracted osTicket into `/var/www/html/osticket`
3. Created a MySQL database and user for osTicket
4. Completed the osTicket web-based installer at `http://<OSTICKET_IP>/osticket`
5. Configured:
   - Admin account credentials
   - Help desk name and email
   - Default department: **SOC Team**
   - Default SLA: **High Priority — 1 hour response**

---

## 4.2 Elastic → osTicket Webhook Integration

The core of this phase was connecting Elastic's alerting system to osTicket via its REST API so that when a detection rule fires, a ticket is automatically created.

### Steps Performed:

**In osTicket:**
1. Generated an **API key** under `Admin Panel > Manage > API`
2. Noted the API endpoint: `http://<OSTICKET_IP>/api/tickets.json`
3. Created a dedicated ticket queue for SIEM alerts: **"SIEM Alerts"**

**In Kibana (Alert Actions):**
1. Opened each detection rule (RDP Brute Force, SSH Brute Force, C2)
2. Under **Actions**, added a **Webhook connector**
3. Configured the connector:
   - **URL:** `http://<OSTICKET_IP>/api/tickets.json`
   - **Method:** POST
   - **Headers:**
     ```
     Content-Type: application/json
     X-API-Key: <OSTICKET_API_KEY>
     ```
   - **Body:**
     ```json
     {
       "name": "SIEM Alert - {{rule.name}}",
       "email": "soc-alerts@lab.local",
       "subject": "[ALERT] {{rule.name}} detected on {{host.name}}",
       "message": "Alert triggered at {{context.date}}\n\nSource IP: {{source.ip}}\nUser: {{user.name}}\nHost: {{host.name}}\nSeverity: {{rule.severity}}\n\nPlease investigate immediately."
     }
     ```

4. Tested the webhook using Kibana's built-in test feature
5. Confirmed a test ticket appeared in osTicket

---

## 4.3 Adversary Simulation #1 — RDP Brute Force

### Attack Setup
The Mythic C2 VM was used as the attacker machine. `crowbar` was used to perform the brute force.

### Attack Execution:
```bash
crowbar -b rdp -s <WINDOWS_TARGET_IP>/32 -u Administrator -C /usr/share/wordlists/rockyou.txt
```

### Detection & Response Timeline:

| Time | Event |
|---|---|
| T+0:00 | RDP brute force begins from `<ATTACKER_IP>` |
| T+0:45 | 10 failed Event ID 4625 events recorded in Elasticsearch |
| T+0:46 | Kibana threshold rule fires |
| T+0:47 | Webhook triggers — osTicket #101 created automatically |
| T+1:00 | SOC analyst (me) opens ticket, begins investigation |
| T+1:15 | Source IP identified and documented in ticket |
| T+1:20 | Ticket resolved — attacker IP added to block list |

### osTicket #101 — Key Details:
- **Subject:** [ALERT] RDP Brute Force detected on Windows-Target
- **Status:** Resolved
- **Root Cause:** Automated brute force from external IP
- **Action Taken:** Source IP documented; Windows firewall rule added to block the IP

---

## 4.4 Adversary Simulation #2 — SSH Brute Force

### Attack Execution:
```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://<UBUNTU_TARGET_IP>
```

### Detection & Response Timeline:

| Time | Event |
|---|---|
| T+0:00 | SSH brute force begins |
| T+0:30 | 10+ failed auth events recorded from same source IP |
| T+0:31 | SSH brute force rule fires in Kibana |
| T+0:32 | osTicket #102 created automatically |
| T+0:50 | Investigation completed — source IP correlated across logs |
| T+1:00 | Ticket resolved — `fail2ban` configured to auto-block future attempts |

---

## 4.5 Adversary Simulation #3 — Mythic C2 Deployment

### Attack Setup
A Mythic C2 payload (Apollo agent) was generated and "executed" on the Windows target to simulate a compromised endpoint.

### Steps:
1. Started the Mythic C2 framework on the attacker VM
2. Generated an Apollo agent payload (HTTP listener)
3. Transferred and executed the payload on the Windows target
4. Confirmed C2 callback appeared in the Mythic dashboard

### Detection & Response:

| Time | Event |
|---|---|
| T+0:00 | Payload executed on Windows target |
| T+0:05 | Sysmon Event ID 3 captured — outbound connection to C2 IP |
| T+0:06 | C2 check-in rule fires in Kibana |
| T+0:07 | osTicket #103 created — Severity: Critical |
| T+0:20 | Process identified via Sysmon Event ID 1 |
| T+0:30 | Process terminated; full log timeline documented in ticket |

### osTicket #103 — Key Details:
- **Subject:** [ALERT] Mythic C2 Agent Check-In detected on Windows-Target
- **Status:** Resolved
- **Root Cause:** Simulated malware executing C2 callback
- **Action Taken:** Malicious process terminated; IOCs documented (C2 IP, process hash, filename)

---

## 4.6 Alert-to-Ticket Lifecycle Summary

The full workflow validated in this phase:

```
Attacker Activity
      │
      ▼
Endpoint Logs (Sysmon / auth.log)
      │
      ▼
Elastic Agent ships logs to Elasticsearch
      │
      ▼
Kibana Detection Rule evaluates logs
      │
      ▼
Alert fires → Webhook triggered
      │
      ▼
osTicket creates Incident Ticket automatically
      │
      ▼
SOC Analyst investigates → Documents findings → Resolves ticket
```

---

## 4.7 Screenshots

> Screenshots for this phase can be found in the [`/screenshots/phase4/`](../screenshots/phase4/) folder.

Key screenshots include:
- osTicket installation complete screen
- Kibana webhook connector configuration
- osTicket ticket #101, #102, #103 showing auto-created alerts
- Mythic C2 dashboard showing active agent callback
- Sysmon event capturing the C2 network connection

---

## 4.8 Issues Encountered & Resolutions

| Issue | Resolution |
|---|---|
| Webhook returning 403 Forbidden from osTicket | Ensured the API key was added to the correct IP whitelist in osTicket admin settings |
| Ticket body showing raw template variables | Used Kibana's Mustache template syntax correctly; verified variable names matched alert context |
| Mythic agent payload flagged by Windows Defender | Disabled Windows Defender on the Windows Target VM for simulation purposes |

---

## Phase 4 Completion Checklist

- [x] osTicket installed and configured with SOC department
- [x] API key generated and webhook connector configured in Kibana
- [x] All detection rules connected to osTicket via webhook
- [x] RDP Brute Force simulation completed — ticket #101 auto-created
- [x] SSH Brute Force simulation completed — ticket #102 auto-created
- [x] Mythic C2 simulation completed — ticket #103 auto-created
- [x] Full Alert → Ticket → Resolve lifecycle documented

---

## Project Complete ✅

This concludes the 30-day MYDFIR SOC Analyst Challenge. Over the course of this project, a fully functional SOC home lab was built from scratch — covering infrastructure provisioning, endpoint telemetry, detection engineering, and automated incident response.

**Skills demonstrated:**
- ELK Stack deployment and configuration
- Endpoint monitoring with Sysmon and Elastic Agent
- Custom detection rule engineering with KQL
- Adversary simulation (Brute Force, C2)
- Automated alert-to-ticket pipeline
- SOC analyst incident response workflow

---

*Previous: [Phase 3 — Detection Engineering & Alerting](phase3-detection-engineering-alerting.md)*  
*Back to: [Project README](../README.md)*
