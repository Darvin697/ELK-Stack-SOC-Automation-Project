# Phase 3 — Detection Engineering & Alerting (Days 14–22)

## Overview

With endpoint telemetry flowing into the SIEM, this phase focused on building custom detection rules using KQL (Kibana Query Language) to identify real attack patterns. Rules were created for RDP Brute Force, SSH Brute Force, and Mythic C2 activity. A SOC dashboard was also built to give a centralized view of security events.

---

## 3.1 Detection Rule: RDP Brute Force

### Objective
Detect when an attacker is attempting to guess credentials over RDP (Remote Desktop Protocol) by generating a high volume of failed login attempts against a Windows endpoint.

### Event Source
Windows Security Event Log — **Event ID 4625** (An account failed to log on)

### Detection Logic
Trigger an alert when more than **10 failed RDP logon attempts** occur from the same source IP within a **5-minute window**.

### KQL Query:
```kql
event.code: "4625" AND winlog.logon.type: "RemoteInteractive"
```

### Rule Configuration in Kibana:
- **Rule Type:** Threshold
- **Threshold Field:** `source.ip`
- **Threshold Count:** 10
- **Time Window:** 5 minutes
- **Severity:** High
- **Actions:** Trigger osTicket webhook

### Alert Fields Included in Notification:
- `source.ip` — Attacker's IP address
- `user.name` — Target username being attacked
- `host.name` — Targeted Windows endpoint
- `@timestamp` — Time of detection

---

## 3.2 Detection Rule: SSH Brute Force

### Objective
Detect brute force attempts against the SSH service on Linux endpoints by monitoring authentication failure logs.

### Event Source
Linux `/var/log/auth.log` — failed SSH authentication events

### Detection Logic
Trigger an alert when more than **10 failed SSH authentication attempts** occur from the same source IP within a **5-minute window**.

### KQL Query:
```kql
event.dataset: "system.auth" AND event.outcome: "failure" AND process.name: "sshd"
```

### Rule Configuration in Kibana:
- **Rule Type:** Threshold
- **Threshold Field:** `source.ip`
- **Threshold Count:** 10
- **Time Window:** 5 minutes
- **Severity:** High
- **Actions:** Trigger osTicket webhook

---

## 3.3 Detection Rule: Mythic C2 Agent Check-In

### Objective
Detect when a Mythic C2 agent installed on a compromised endpoint checks in with the C2 server, indicating an active post-exploitation session.

### Event Source
Sysmon **Event ID 3** (Network Connection) on Windows endpoint

### Detection Logic
Alert on outbound network connections from uncommon processes to the known Mythic C2 server IP on unusual ports.

### KQL Query:
```kql
event.dataset: "windows.sysmon_operational" 
AND event.code: "3" 
AND destination.ip: "<MYTHIC_C2_IP>"
```

> **Note:** The Mythic C2 server IP was known in the lab environment. In a real SOC, this rule would be enriched with threat intelligence feeds.

### Rule Configuration in Kibana:
- **Rule Type:** Query (Custom Query)
- **Severity:** Critical
- **Actions:** Trigger osTicket webhook + email notification

---

## 3.4 Rule Testing & Tuning

After writing each rule, simulations were run to validate they fired correctly:

1. **RDP Brute Force test:** Used `crowbar` or manual failed RDP login attempts from the Mythic C2 VM against the Windows target. Confirmed alert triggered after threshold was met.

2. **SSH Brute Force test:** Used `hydra` to generate failed SSH login attempts against the Ubuntu target. Confirmed alert triggered.

3. **False Positive tuning:** During testing, some rules fired on legitimate admin activity. The rules were tuned by:
   - Adding exceptions for known admin IPs
   - Raising thresholds where necessary
   - Filtering on specific logon types

---

## 3.5 SOC Dashboard

A custom Kibana dashboard was created to give a real-time, unified view of security events across the environment.

### Dashboard Panels:

| Panel | Visualization Type | Data Source |
|---|---|---|
| Failed Login Attempts (Last 24h) | Metric | Windows + Linux auth logs |
| Failed Logins by Source IP | Data Table | Windows Event ID 4625 + auth.log |
| Global Attack Map | Maps (Geo) | `source.geo` fields |
| Top Targeted Usernames | Bar Chart | `user.name` field |
| Active Alerts by Severity | Donut Chart | `.alerts-*` index |
| Timeline of Security Events | Line Chart | All log sources |

### Dashboard Purpose:
The dashboard simulates what a SOC analyst would monitor during a shift — giving immediate visibility into where attacks are coming from, which accounts are being targeted, and how many active alerts need investigation.

---

## 3.6 Screenshots

> Screenshots for this phase can be found in the [`/screenshots/phase3/`](../screenshots/phase3/) folder.

Key screenshots include:
- KQL rule configuration in Kibana for RDP brute force
- Alert firing after brute force simulation
- SOC dashboard with live data
- Global attack map showing source IPs

---

## 3.7 Issues Encountered & Resolutions

| Issue | Resolution |
|---|---|
| RDP brute force rule not firing | Event ID 4625 was present but logon type filter was too restrictive — removed the `RemoteInteractive` filter to broaden capture |
| Mythic C2 rule generating too many alerts | Added a filter to only alert on specific processes known to be used by Mythic agents |
| Dashboard geo map not showing IPs | Enabled GeoIP enrichment processor in the Elasticsearch ingest pipeline |

---

## Phase 3 Completion Checklist

- [x] RDP Brute Force detection rule created and tested
- [x] SSH Brute Force detection rule created and tested
- [x] Mythic C2 check-in detection rule created and tested
- [x] All rules validated with live attack simulations
- [x] False positives tuned and minimized
- [x] SOC Dashboard created with all key panels
- [x] Alerts configured to trigger osTicket webhook

---

*Previous: [Phase 2 — Telemetry & Log Ingestion](phase2-telemetry-log-ingestion.md)*  
*Next: [Phase 4 — Incident Response & Ticketing](phase4-incident-response-ticketing.md)*
