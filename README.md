# ELK-Stack-SOC-Automation-Project

This project demonstrates the end-to-end setup and management of a Security Operations Center (SOC) environment deployed on Vultr Cloud.
integrating  SIEM capabilities with Endpoint Detection and Response (EDR) and an automated ticketing system (OS Ticket).
It features real-world attack simulations to validate detection rules and incident response workflows.(SSH,RDP Bruteforce)


ELK Stack SOC Automation Project

🛡️ Project Overview
This project demonstrates the end-to-end setup and management of a Security Operations Center (SOC) environment. I built a functional home lab that integrates SIEM capabilities with endpoint monitoring and an automated ticketing system to detect and respond to real-world attack simulations.

🎯 Key Objectives
Deploy a full ELK Stack (Elasticsearch, Kibana, Fleet Server).

Configure endpoint telemetry using Elastic Agent and Sysmon on Windows and Ubuntu.

Engineer custom detection rules for Brute Force (SSH/RDP) and Command & Control (C2) traffic.

Integrate osTicket to automate the transition from "Alert" to "Incident Ticket."

🏗️ Architecture



The environment consists of:

SIEM Server: Ubuntu 24.04 running the Elastic Stack.

Endpoints: Windows Server 2022 (Target) & Ubuntu Server (Target).

Attack Platform: Mythic C2 Framework.

Case Management: osTicket integrated via API/Webhooks.

🚀 Implementation Phases
Phase 1: Infrastructure & SIEM Setup (Days 1–7)
Provisioned virtualized environments for SIEM and Endpoints.

Configured Elasticsearch for data storage and Kibana for visualization.

Established a Fleet Server for centralized agent management.

Phase 2: Telemetry & Ingestion (Days 8–13)
Deployed Sysmon on Windows for granular event logging (Process creation, Network connections).

Installed Elastic Agents on Linux and Windows to ship logs to the SIEM.

Normalized data using Elastic Common Schema (ECS).

Phase 3: Detection Engineering & Alerting (Days 14–22)
Built custom KQL (Kibana Query Language) alerts for:

Successful/Failed RDP Brute Force.

SSH Authentication failures.

Mythic Agent C2 check-ins.

Created a unified SOC Dashboard to visualize global login attempts and system health.

Phase 4: Incident Response & Ticketing (Days 23–30)
Configured osTicket to receive alerts from Elastic.

Conducted Adversary Simulations (SSH/RDP Brute Force).

Documented the full "Alert-to-Ticket" lifecycle, simulating a professional SOC Analyst workflow.

📊 Investigation Examples
Attack Type	Detection Method	Mitigation/Response
RDP Brute Force	Threshold Alert (>10 failed logins)	Identified Source IP, Created osTicket #102
C2 Communication	Mythic Agent Heartbeat Detection	Log Analysis, Process Termination
🛠️ Tools Used
SIEM: Elastic Stack

EDR/Logs: Sysmon, Elastic Agent

Ticketing: osTicket

C2: Mythic

Cloud/VMs: [Insert your platform, e.g., VirtualBox, VMWare, or Azure]

Next Step for your GitHub:
Would you like me to show you how to write the Investigation Report for Day 26 (SSH Brute Force) to include as a PDF or Markdown file in your repo?
