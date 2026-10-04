# 🌐 SOC Home Lab Network Architecture

## 🖥️ Network Configuration

Each VM has **two Network Adapters** in VMware:

| Adapter | Type | Purpose |
|---|---|---|
| Adapter 1 | NAT | Internet access (OS updates, tool downloads) |
| Adapter 2 | Host-Only (VMnet1) | Internal communication between VMs (logs & attacks) |

### IP Addressing

| VM | IP Address |
|---|---|
| Ubuntu Server | `192.168.56.10/24` |
| Windows 10 | `192.168.56.30/24` |
| Kali Linux | `192.168.56.105/24` |

> **Host-Only Gateway:** `192.168.56.1` (VMware default)

---

## 📨 Log Flow

1. **Windows 10** records security events to the *Windows Event Log* (Security, System, Application) and system activity via **Sysmon** (Process Creation, Network Connection, etc.).
2. **Splunk Universal Forwarder (UF)** on Windows 10 reads those logs using the `inputs.conf` configuration and sends them to **Ubuntu Server** on port `9997`.
3. **Splunk Enterprise** on Ubuntu Server receives, indexes, and stores the logs in the `windows` and `sysmon` indexes.
4. **SOC Analyst L1** accesses the Splunk Web UI (`http://192.168.56.10:8000`) for monitoring, searching, triage, and investigation.

---

## ⚔️ Simulated Attack Scenarios

| Live Attack | Description | Event ID / Sysmon ID | MITRE ATT&CK |
|---|---|---|---|
| **1. Brute Force** | Hydra attacks RDP/SMB on Windows 10 | `4625` (Logon Type 3) | T1110 |
| **2. Phishing** | Simulated malicious email leads to PowerShell execution | Sysmon `1` | T1566 |
| **3. Suspicious PowerShell** | Encoded PowerShell command execution | Sysmon `1`, Windows `4104` | T1059.001 |

---

## 🔐 Security Notes

- ⚠️ This lab is **for educational purposes only** and runs in an isolated environment.
- 🚫 Never expose these VMs to public networks.
- 🔥 Disable Windows Firewall temporarily only during simulations.
- 📸 Always take **Snapshots** before and after major changes.

---

## 📂 Related Documentation

- [Splunk UF Configuration](../02-Configuration/inputs.conf)
- [Splunk Detection Queries](../04-Detection-Splunk/splunk-queries.md)
- [Incident Report INC-2026-001](../05-Incident-Reports/INC-2026-001-BruteForce.md)

---

## 🎯 Conclusion

This architecture gives a complete picture of how a small-scale SOC is built, from endpoint log collection to SIEM ingestion and analyst investigation. Understanding this flow provides a strong foundation for a career as a **SOC Analyst Level 1**.

---

*This document is part of the [SOC Home Lab - Splunk & Sysmon](https://github.com/adiirmdhn/SOC-Home-Lab-Splunk) project.*
