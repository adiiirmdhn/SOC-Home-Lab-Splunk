# Live Attack 1 - Brute Force Simulation

## 🎯 Objective
Simulate a Brute Force attack against the RDP service on Windows 10 to test the detection capabilities of Sysmon and Splunk SIEM.

---

## 🛠️ Tool
- **Hydra** (Kali Linux)

---

## 💻 Command
```bash
hydra -l testuser -P /usr/share/wordlists/rockyou.txt -t 1 -W 3 rdp://192.168.56.30
```

Parameter explanation:

- `-l testuser` : Target username
- `-P /usr/share/wordlists/rockyou.txt` : Wordlist file containing password candidates
- `-t 1` : Use 1 thread to avoid triggering rate-limiting
- `-W 3` : Wait 3 seconds between connections
- `rdp://192.168.56.30` : Target IP and protocol

---

## 📊 Result
Hydra failed to guess the password (as expected), but it successfully triggered multiple Event ID 4625 (Failed Login) on Windows 10.

- Number of attempts: 9 failed logins
- Logon Type: 3 (Network Logon)
- Workstation Name: kali

---

## 🔍 Detection
The attack was detected in Splunk using the following query:

```spl
index=windows EventCode=4625
| stats count by Source_Network_Address, Account_Name
| where count > 5
```

The detection results showed a spike in failed logins from IP `192.168.56.105` (Kali Linux).

---

## 🧭 MITRE ATT&CK Mapping
- **Tactic:** Credential Access
- **Technique:** T1110 - Brute Force

---

## 📌 Notes
Although Hydra did not successfully guess the password, the attack was successfully detected and recorded in Splunk. This highlights the importance of log correlation between Windows Event Log and Sysmon to detect suspicious activity.

Part of the [SOC Home Lab - Splunk & Sysmon](https://github.com/adiirmdhn/SOC-Home-Lab-Splunk) project.
