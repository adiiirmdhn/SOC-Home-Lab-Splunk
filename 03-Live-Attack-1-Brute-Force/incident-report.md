# Incident Report - INC-2026-001

## Summary

| Field | Value |
|---|---|
| Incident ID | INC-2026-001 |
| Title | Brute Force Attempt - RDP (testuser) |
| Severity | Medium |
| Status | Escalated to Tier 2 |
| Date detected | 2026-10-02 14:29:14 (UTC+7) |
| Analyst | Adi Ramadhani |
| Related lab | Live Attack 1 - Brute Force ([README](README.md)) |

---

## Detection Source

- **Tool:** Splunk SIEM
- **Log source:** Windows Security Event Log (forwarded via Sysmon)
- **Query used:**

```spl
index=windows EventCode=4625
| stats count by Source_Network_Address, Account_Name
| where count > 5
```

---

## What Happened

13 instances of Event ID 4625 (failed logon) were logged in a short window, all Logon Type 3 (network logon), all originating from host `kali` at 192.168.56.105, all targeting the local account `testuser` on the Windows 10 endpoint (192.168.56.30).

No successful authentication was recorded for the target account during or after the attempt window.

| Attribute | Value |
|---|---|
| Source IP | 192.168.56.105 |
| Source hostname | kali |
| Target account | testuser |
| Target host | 192.168.56.30 (Windows 10) |
| Logon type | 3 (Network) |
| Failed attempts | 13 |
| Successful logon | None |

---

## Analysis

The pattern of repeated failed logons against a single account, from a single source, in a tight time window, is consistent with an automated brute force attempt rather than a user mistyping a password. Volume and timing rule out normal user error.

**MITRE ATT&CK mapping**
- Tactic: Credential Access
- Technique: T1110 - Brute Force

---

## Actions Taken

- Escalated to Tier 2 for further investigation
- Recommended blocking source IP 192.168.56.105 at the firewall
- Recommended enabling account lockout policy on affected systems
- Flagged `testuser` for continued monitoring over the following 48 hours

---

## Verdict

**True Positive** - confirmed brute force attempt. No account compromise occurred.

---

## Lessons Learned

Account lockout policy was not enforced on the target host, which let the attempt run to 13 tries without any automatic block. Enabling lockout after a low threshold (e.g. 5 failed attempts) would reduce the window for this kind of attack and should be checked across other endpoints in the lab.
