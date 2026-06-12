# splunk-siem-home-lab
A hands-on cybersecurity home lab simulating a real-world SOC environment — built from scratch to detect network reconnaissance and brute force attacks using Splunk Enterprise, Kali Linux, and Windows 10.

> **Goal:** Set up Splunk Enterprise as a SIEM, configure a Windows Universal Forwarder, simulate Nmap scans and RDP brute force attacks from Kali, write SPL detection rules, configure alerts, and build a SOC analyst dashboard.
 
---
 
## 🗺️ Lab Architecture
 
```
┌──────────────────────────────────────────────────────┐
│                 VMware NAT Network                   │
│                 192.168.23.0/24                      │
│                                                      │
│  ┌──────────────┐         ┌──────────────────────┐   │
│  │  Kali Linux  │───────▶ │     Windows 10       │  │  
│  │ (Attacker)   │  Attack │  Universal Forwarder │   │
│  └──────────────┘         └──────────┬───────────┘   │
│                                      │ Logs (9997)   │
│        ┌──────────▼───────────┐   │──│               │
│        │   Ubuntu Server      │___│                  │
│        │  Splunk Enterprise   │                      │
│        └──────────────────────┘                      │
└──────────────────────────────────────────────────────┘
```
 
---
 
## 🛠️ Tools & Technologies
 
| Tool | Purpose |
|------|---------|
| Splunk Enterprise 10.4 | SIEM — log ingestion, detection, alerting, dashboards |
| Splunk Universal Forwarder | Forward Windows Security logs to Splunk |
| Kali Linux | Attacker machine (Nmap, Hydra) |
| Nmap | Network reconnaissance / port scanning |
| Hydra | RDP brute force simulation |
| Windows 10 | Target machine with forwarder installed |
| Metasploitable2 | Vulnerable Linux target (lab completeness) |
| VMware Workstation | Virtualization platform |
 
---
 
## 📋 Project Walkthrough
 
### Phase 1 — Environment Setup
 
**1. Ubuntu Server Update**
Updated and upgraded Ubuntu Server packages before installation.
![Ubuntu Update](screenshots/ubuntu-update.png)
 
**2. Ubuntu Network Configuration**
Verified Ubuntu Server's IP address on the lab network.
![Ubuntu IP](screenshots/ubuntu-ip.png)
 
---
 
### Phase 2 — Splunk Enterprise Installation
 
**3. Splunk Installation**
Installed Splunk Enterprise on Ubuntu Server.
![Splunk Installation](screenshots/splunk-installation.png)
 
**4. Admin User Setup**
Configured the Splunk administrator username and password during first-time setup.
![User Setup](screenshots/splunk-user-and-password-setup.png)
 
**5. Splunk Login Page**
Splunk Web UI accessible at `http://<ubuntu-ip>:8000`.
![Login Page](screenshots/splunk-login-page.png)
 
**6. Splunk Home Page**
Successfully logged into Splunk Enterprise dashboard.
![Home Page](screenshots/splunk-home-page.png)
 
---
 
### Phase 3 — Configuring Splunk to Receive Logs
 
**7. Receiving Ports Configuration**
Enabled receiving on **TCP port 9997** (forwarder data) and **514** (syslog).
![Receiving Ports](screenshots/configured-recieve-ports.png)
 
---
 
### Phase 4 — Windows Forwarder Setup
 
**8. Splunk Universal Forwarder Installation**
Installed Universal Forwarder on the Windows 10 target machine.
![Forwarder Installation](screenshots/splunk-forwarder-installation.png)
 
**9. inputs.conf Configuration**
Configured `inputs.conf` to monitor Windows Security event logs.
![inputs.conf](screenshots/inputs.conf-configuration.png)
 
**10. outputs.conf Configuration**
Configured `outputs.conf` to point the forwarder to the Splunk indexer (Ubuntu IP, port 9997).
![outputs.conf](screenshots/outputs.conf-configuration.png)
 
**11. Restarting Splunk Forwarder**
Restarted the forwarder service to apply configuration changes.
![Restarting Forwarder](screenshots/restarting-splunk-forwarder.png)
 
**12. Verifying Windows Logs in Splunk**
Confirmed Windows Security logs (`host=win-ad`) flowing into Splunk's `main` index.
![Verifying Logs](screenshots/verifying-windows-logs-on-splunk.png)
 
---
 
### Phase 5 — Attack Simulation 1: Network Reconnaissance
 
**13. Nmap Scan on Windows**
Ran an aggressive Nmap scan from Kali against the Windows 10 target to enumerate open ports and services.
```bash
nmap -sS -A 192.168.23.129
```
![Nmap Scan](screenshots/nmap-scan-to-windows.png)
 
**14. Nmap Scan Detected in Splunk**
Scan activity reflected in Windows event logs, captured by Splunk.
![Nmap Alert](screenshots/nmap-scan-alert.png)
 
---
 
### Phase 6 — Attack Simulation 2: RDP Brute Force
 
**15. Brute Force Attack on Windows**
Launched a brute force attack against the Windows RDP service (port 3389) using Hydra.
```bash
hydra -l administrator -P /usr/share/wordlists/rockyou.txt \
  rdp://192.168.23.129 -t 4 -V -f
```
![Brute Force Attack](screenshots/brute-force-attack-on-windows.png)
 
**16. Brute Force Detected in Splunk**
Splunk captured `EventCode 4625` (failed logon) events with the attacker's account name, workstation, and source IP.
![Brute Force Alert](screenshots/brute-force-alert-01.png)
![Brute Force Alert](screenshots/brute-force-alert-02.png)
 
---
 
### Phase 7 — Detection & Alerting
 
**17 & 18. Saving the Detection Alert**
Created and saved a scheduled alert based on the SPL detection query, configured to trigger when failed login count exceeds threshold.
![Saving Alert 1](screenshots/saving-alert-01.png)
![Saving Alert 2](screenshots/saving-alert-02.png)
 
---
 
### Phase 8 — Building the SOC Dashboard
 
**19. Creating the Dashboard**
Created a new Classic Dashboard named **"SOC Home Lab"**.
![Creating Dashboard](screenshots/creating-dashboard.png)
 
**20–23. Adding Dashboard Panels**
Built four panels to visualize attack activity:
 
| Panel | Type | Purpose |
|-------|------|---------|
| Failed Logins Over Time | Line Chart | Shows spike during brute force attack |
| Top Targeted Accounts | Bar Chart | Highlights `administrator` as the targeted account |
| Attacker Source IP | Statistics Table | Identifies attacker IP/workstation |
| Login Success vs Failed | Column Chart | Compares normal vs failed login activity |
 
![Panel 1](screenshots/creating-panel-01.png)
![Panel 2](screenshots/creating-panel-02.png)
![Panel 3](screenshots/creating-panel-03.png)
![Panel 4](screenshots/creating-panel-04.png)
 
**24 & 25. Final SOC Dashboard**
Completed dashboard with all panels showing live attack data.
![Dashboard 1](screenshots/dashboard-01.png)
![Dashboard 2](screenshots/dashboard-02.png)
 
---
 
## 🔍 SPL Detection Queries
 
All queries used in this project are documented in [`spl-queries/detection-rules.spl`](spl-queries/detection-rules.spl)
 
### Brute Force Detection
```spl
index="main" host="win-ad" EventCode=4625
| stats count by Account_Name
| where count > 10
| sort -count
```
 
### Failed Logins Over Time
```spl
index="main" host="win-ad" EventCode=4625
| timechart count span=5m
```
 
### Attacker IP Identification
```spl
index="main" host="win-ad" EventCode=4625
| stats count by src_ip
| sort -count
```
 
### Login Success vs Failed
```spl
index="main" host="win-ad" (EventCode=4624 OR EventCode=4625)
| eval Status=if(EventCode=4624,"Success","Failed")
| timechart count by Status span=5m
```
 
### Top Targeted Accounts
```spl
index="main" host="win-ad" EventCode=4625
| top limit=5 Account_Name
```
 
---
 
## 🚨 Alert Configuration
 
| Field | Value |
|-------|-------|
| Alert Name | Brute Force - RDP Failed Logins |
| Search | EventCode=4625, stats by Account_Name |
| Trigger Condition | Number of results > 0 |
| Schedule | Every 5 minutes (`*/5 * * * *`) |
| Severity | Medium |
| Action | Add to Triggered Alerts |
 
---
 
## 📁 Repository Structure
 
```
splunk-home-lab/
├── README.md
├── screenshots/
│   ├── ubuntu-update.png
│   ├── ubuntu-ip.png
│   ├── splunk-installation.png
│   ├── splunk-user-and-password-setup.png
│   ├── splunk-login-page.png
│   ├── splunk-home-page.png
│   ├── configured-recieve-ports.png
│   ├── splunk-forwarder-installation.png
│   ├── inputs.conf-configuration.png
│   ├── outputs.conf-configuration.png
│   ├── restarting-splunk-forwarder.png
│   ├── verifying-windows-logs-on-splunk.png
│   ├── nmap-scan-to-windows.png
│   ├── nmap-scan-alert.png
│   ├── brute-force-attack-on-windows.png
│   ├── brute-force-alert-01.png
│   ├── brute-force-alert-02.png
│   ├──saving-alert-01.png
│   ├── saving-alert-02.png
│   ├── creating-dashboard.png
│   ├── creating-panel-01.png
│   ├── creating-panel-02.png
│   ├── creating-panel-03.png
│   ├── creating-panel-04.png
│   ├── dashboard-01.png
│   └── dashboard-02.png
├──spl-queries/
│   └── detection-rules.spl
└── setup/
    └── lab-topology.md
```
 
---
 
## 🧠 Key Learnings
 
- Installed and configured Splunk Enterprise as a centralized SIEM on Ubuntu Server
- Deployed Splunk Universal Forwarder on Windows 10 and configured `inputs.conf` / `outputs.conf` for log forwarding over TCP 9997
- Verified end-to-end log ingestion from Windows Security event logs
- Simulated real-world attacker behavior: network reconnaissance (Nmap) and credential brute forcing (Hydra) against RDP
- Identified attack signatures in Windows Security logs (`EventCode 4625`, `EventCode 4634`)
- Wrote custom SPL queries to detect brute force patterns and identify attacker source IPs
- Configured a scheduled Splunk alert that successfully triggered on attack detection
- Built a 4-panel SOC dashboard for real-time monitoring of authentication activity
---
 
## 👤 Author
 
**Mukul**  
Cybersecurity Student
 
[![GitHub](https://github.com/mukulkumar-labs)
[![LinkedIn](https://www.linkedin.com/in/mukul-kumar-31b54a408/)
 
---
 
## ⚠️ Disclaimer
 
This project was built entirely in an isolated virtual lab environment for educational purposes. All attacks were performed against machines I own and control. These techniques must never be used against systems without explicit authorization.
