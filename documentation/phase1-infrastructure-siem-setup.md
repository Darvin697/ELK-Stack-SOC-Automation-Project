# Phase 1 — Infrastructure & SIEM Setup (Days 1–7)

## Overview

This phase covers the initial provisioning of all cloud infrastructure on Vultr and the full deployment of the ELK Stack (Elasticsearch, Kibana, and Fleet Server). By the end of this phase, the SIEM was fully operational and ready to receive logs from endpoints.

---

## 1.1 Cloud Infrastructure Provisioning

### Provider: Vultr

All virtual machines were provisioned on the Vultr cloud platform. The following VMs were created:

| VM Name | OS | Purpose | Specs |
|---|---|---|---|
| ELK-SIEM | Ubuntu 24.04 LTS | Elasticsearch + Kibana | 4 vCPU / 8GB RAM |
| Fleet-Server | Ubuntu 24.04 LTS | Elastic Fleet Server | 2 vCPU / 4GB RAM |
| Windows-Target | Windows Server 2022 | Endpoint (target machine) | 2 vCPU / 4GB RAM |
| Ubuntu-Target | Ubuntu 24.04 LTS | Endpoint (target machine) | 2 vCPU / 4GB RAM |
| Mythic-C2 | Ubuntu 24.04 LTS | Attack platform | 2 vCPU / 4GB RAM |
| osTicket-Server | Ubuntu 24.04 LTS | Case management | 2 vCPU / 4GB RAM |

### Network Configuration
- All VMs were placed within the same Vultr private network for internal communication
- Firewall rules were configured to restrict public access to only necessary ports
- SSH key-based authentication was enforced on all Linux VMs

---

## 1.2 Elasticsearch Installation

Elasticsearch was installed on the SIEM VM as the core data storage and search engine.

### Steps Performed:
1. Imported the Elastic GPG key and added the Elastic APT repository
2. Installed Elasticsearch via `apt`
3. Configured `/etc/elasticsearch/elasticsearch.yml`:
   - Set `network.host` to the VM's private IP
   - Configured `cluster.name` and `node.name`
   - Enabled security features (TLS + authentication)
4. Generated SSL certificates using `elasticsearch-certutil`
5. Started and enabled the Elasticsearch service
6. Verified cluster health via the REST API

### Key Configuration Snippet:
```yaml
cluster.name: soc-lab
node.name: siem-node-1
network.host: <PRIVATE_IP>
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
```

---

## 1.3 Kibana Installation

Kibana was installed on the same SIEM VM to provide the visualization and management interface.

### Steps Performed:
1. Installed Kibana via `apt`
2. Configured `/etc/kibana/kibana.yml`:
   - Set `server.host` to `0.0.0.0` to allow external access
   - Pointed `elasticsearch.hosts` to the local Elasticsearch instance
   - Configured `elasticsearch.username` and `elasticsearch.password`
3. Generated a Kibana service account token
4. Started and enabled the Kibana service
5. Accessed the Kibana web UI and completed initial setup

### Verification:
- Navigated to `http://<SIEM_IP>:5601` in browser
- Logged in with the `elastic` superuser account
- Confirmed the Kibana status page showed green

---

## 1.4 Fleet Server Setup

The Fleet Server was deployed on a dedicated VM to act as a proxy between Elastic Agents on endpoints and the Elasticsearch cluster.

### Steps Performed:
1. Downloaded the Elastic Agent package on the Fleet Server VM
2. In Kibana, navigated to **Fleet > Settings** and added the Fleet Server host URL
3. Generated a Fleet Server enrollment token
4. Ran the Elastic Agent install command with the `--fleet-server-es` flag to bootstrap it as the Fleet Server
5. Confirmed the Fleet Server appeared as **Healthy** in the Kibana Fleet dashboard

### Command Used:
```bash
sudo ./elastic-agent install \
  --fleet-server-es=https://<SIEM_IP>:9200 \
  --fleet-server-service-token=<TOKEN> \
  --fleet-server-policy=fleet-server-policy \
  --fleet-server-es-ca-trusted-fingerprint=<FINGERPRINT>
```

---

## 1.5 Screenshots

> Screenshots for this phase can be found in the [`/screenshots/phase1/`](../screenshots/phase1/) folder.

Key screenshots include:
- Vultr VM dashboard showing all provisioned machines
- Elasticsearch cluster health API response
- Kibana initial login and home screen
- Fleet Server showing as Healthy in Fleet UI

---

## 1.6 Issues Encountered & Resolutions

| Issue | Resolution |
|---|---|
| Kibana could not connect to Elasticsearch | Corrected the `elasticsearch.hosts` value to use HTTPS and the correct fingerprint |
| Fleet Server enrollment token expired | Regenerated a new token from the Kibana Fleet settings |
| Firewall blocking port 5601 | Added inbound rule for port 5601 on the Vultr firewall group |

---

## Phase 1 Completion Checklist

- [x] All VMs provisioned on Vultr
- [x] Elasticsearch installed, secured, and running
- [x] Kibana installed and accessible via browser
- [x] Fleet Server enrolled and showing Healthy
- [x] Network and firewall rules configured
- [x] SSL/TLS enabled on all Elastic components

---

*Next: [Phase 2 — Telemetry & Log Ingestion](phase2-telemetry-log-ingestion.md)*
