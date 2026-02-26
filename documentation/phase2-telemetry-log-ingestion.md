# Phase 2 — Telemetry & Log Ingestion (Days 8–13)

## Overview

With the SIEM stack running, this phase focused on deploying monitoring agents to all endpoint VMs and configuring them to ship logs to Elasticsearch. Sysmon was deployed on the Windows target for deep endpoint visibility, while Elastic Agent was installed on both Windows and Linux endpoints.

---

## 2.1 Sysmon Deployment on Windows Server 2022

Sysmon (System Monitor) is a Windows system service that logs detailed process, network, and file activity to the Windows Event Log — far beyond what standard Windows logging provides.

### Steps Performed:
1. Downloaded Sysmon from the [Microsoft Sysinternals](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon) page
2. Downloaded the **SwiftOnSecurity Sysmon config** (`sysmonconfig.xml`) — a well-maintained, community-tuned configuration
3. Installed Sysmon with the config file:
```powershell
.\Sysmon64.exe -accepteula -i sysmonconfig.xml
```
4. Verified Sysmon service was running in `services.msc`
5. Confirmed events appearing in **Event Viewer** under `Applications and Services Logs > Microsoft > Windows > Sysmon`

### Key Sysmon Event IDs Captured:

| Event ID | Description |
|---|---|
| 1 | Process Creation |
| 3 | Network Connection |
| 7 | Image Loaded (DLL) |
| 10 | Process Access |
| 11 | File Create |
| 22 | DNS Query |

---

## 2.2 Elastic Agent Deployment — Windows Target

The Elastic Agent was enrolled into Fleet to ship both Windows Event Logs (including Sysmon) and system metrics to Elasticsearch.

### Steps Performed:
1. In Kibana Fleet, created a new **Agent Policy** called `Windows-Endpoints`
2. Added the **Windows integration** to the policy (captures Security, System, Sysmon logs)
3. Added the **System integration** for host metrics (CPU, memory, disk)
4. Downloaded the Elastic Agent installer for Windows
5. Ran the enrollment command in PowerShell (as Administrator):
```powershell
.\elastic-agent.exe install `
  --url=https://<FLEET_SERVER_IP>:8220 `
  --enrollment-token=<TOKEN> `
  --certificate-authorities=<CA_PATH>
```
6. Verified the agent appeared as **Healthy** in Fleet → Agents

### Integrations Enabled:
- Windows Event Logs (Security, System, Application)
- Sysmon logs (via Windows integration custom path)
- System metrics

---

## 2.3 Elastic Agent Deployment — Ubuntu Target

The Linux target was configured to ship authentication logs, syslog, and system metrics.

### Steps Performed:
1. In Kibana Fleet, created a new **Agent Policy** called `Linux-Endpoints`
2. Added the **System integration** and **Linux integration**
3. Downloaded the Elastic Agent `.tar.gz` for Linux
4. Ran the enrollment command:
```bash
sudo ./elastic-agent install \
  --url=https://<FLEET_SERVER_IP>:8220 \
  --enrollment-token=<TOKEN> \
  --certificate-authorities=<CA_PATH>
```
5. Verified agent appeared as **Healthy** in Fleet

### Log Sources Captured:
- `/var/log/auth.log` — SSH authentication events
- `/var/log/syslog` — System events
- System metrics (CPU, memory, network)

---

## 2.4 Elastic Common Schema (ECS) Normalization

All ingested logs were automatically normalized by the Elastic Agent integrations to follow the **Elastic Common Schema (ECS)** — a standardized field naming convention that makes it easier to write detection rules and correlate events across different data sources.

### Key ECS Fields Used Later in Detection Rules:

| ECS Field | Description |
|---|---|
| `event.action` | The action that triggered the event (e.g., `logon-failed`) |
| `source.ip` | The source IP address of a network connection |
| `user.name` | The username involved in the event |
| `process.name` | The name of the process that triggered the event |
| `host.name` | The hostname where the event originated |

---

## 2.5 Verifying Data in Kibana

After agent deployment, data ingestion was verified in Kibana:

1. Navigated to **Discover** in Kibana
2. Selected the `logs-*` and `metrics-*` data views
3. Confirmed logs streaming from both Windows and Linux endpoints
4. Ran a test query to isolate Sysmon events:
```kql
event.dataset: "windows.sysmon_operational"
```

---

## 2.6 Screenshots

> Screenshots for this phase can be found in the [`/screenshots/phase2/`](../screenshots/phase2/) folder.

Key screenshots include:
- Sysmon installed and running in Windows Services
- Sysmon events visible in Event Viewer
- Fleet dashboard showing both agents Healthy
- Kibana Discover showing live log ingestion from both endpoints

---

## 2.7 Issues Encountered & Resolutions

| Issue | Resolution |
|---|---|
| Elastic Agent on Windows couldn't reach Fleet Server | Added Fleet Server IP to Windows Firewall inbound exceptions; verified port 8220 open |
| Sysmon logs not appearing in Kibana | Added the correct Sysmon event log path in the Windows integration settings |
| Linux agent enrollment failed with certificate error | Copied the Elasticsearch CA cert to the Linux VM and referenced it with `--certificate-authorities` |

---

## Phase 2 Completion Checklist

- [x] Sysmon installed on Windows target with SwiftOnSecurity config
- [x] Elastic Agent enrolled on Windows target — showing Healthy
- [x] Elastic Agent enrolled on Ubuntu target — showing Healthy
- [x] Windows Event Logs (Security, Sysmon, System) flowing into Elasticsearch
- [x] Linux auth logs and syslog flowing into Elasticsearch
- [x] Data verified in Kibana Discover

---

*Previous: [Phase 1 — Infrastructure & SIEM Setup](phase1-infrastructure-siem-setup.md)*  
*Next: [Phase 3 — Detection Engineering & Alerting](phase3-detection-engineering-alerting.md)*
