# SOC Investigation Report — SOC176: RDP Brute Force Detected

---

**Information:**

| Field | Detail |
|---|---|
| Event Time | Mar 07, 2024, 11:44 AM |
| Severity | High |
| Rule | SOC176 — RDP Brute Force Detected |

---

## Executive Summary

The SIEM triggered a high-severity alert for RDP Brute Force activity targeting an internal endpoint. Investigation determined that an external attacker launched multiple RDP login attempts using various non-existent accounts, ultimately resulting in a successful login. Post-compromise EDR analysis revealed the execution of multiple suspicious reconnaissance commands. The host was successfully isolated to contain the breach, and the incident has been escalated for eradication and lateral movement analysis.

---

## Alert Details

| Field | Detail |
|---|---|
| Rule Triggered | SOC176 — RDP Brute Force Detected |
| Log Source | Network logs, EDR |
| Source Address | 218.92.0.56 (External attacker) |
| Destination Address | 172.16.17.148 (Internal host: Matthew) |
| Observation | 30 network occurrences showing multiple failed RDP attempts, followed by one successful connection. |
| Status | Closed — Host Isolated / Escalated for Remediation |

---

## Investigation Timeline

| Timestamp | Action |
|---|---|
| 2024-03-07 11:44 | Alert triggered — SOC176: RDP Brute Force Detected. |
| 2024-03-07 11:44 | Initial triage — Conducted IP enrichment. Source IP is an external Chinese telecommunication services IP, flagged by 9 security vendors on VirusTotal. |
| 2024-03-07 11:44 | Network traffic analysis — Observed 30 RDP occurrences communicating with 1 internal host. Identified multiple failed login attempts followed by one successful login attempt. |
| 2024-03-07 11:44 | Checked email logs to determine if this was authorized or planned activity (e.g., admin work). No relevant emails found. |
| 2024-03-07 11:45 | Pivoted to EDR due to successful login. Began analyzing system activity for post-compromise behavior. |
| 2024-03-07 11:45 | EDR flagged suspicious command-line execution: `"C:\Windows\system32\cmd.exe"` (11:45:18), `whoami` (11:45:51), `net user letsdefend` (11:45:58). |
| 2024-03-07 11:46 | Continued suspicious command execution: `net localgroup administrators` (11:46:34), `netstat -ano` (11:46:53). |
| 2024-03-07 11:48 | Confirmed breach based on anomalous reconnaissance commands. Initiated endpoint isolation to contain the threat. |

---

## Findings

An external attacker operating from a Chinese-registered IP address (218.92.0.56) conducted an RDP brute force attack against an internal host (172.16.17.148 / Matthew). After multiple failed attempts with non-existent accounts, the attacker successfully breached the system. Post-exploitation, the attacker launched a command prompt and executed local discovery and enumeration commands (`whoami`, `net user`, `net localgroup`, `netstat`). The host processes did not show any glaringly suspicious malware payloads, indicating the attacker was in the initial reconnaissance phase. 

---

## MITRE ATT&CK Mapping

| Technique | ID | Description |
|---|---|---|
| Brute Force | T1110 | Attacker used automated or manual attempts to guess credentials to gain access. |
| Password Guessing | T1110.001 | Attacker repeatedly guessed passwords for various accounts over RDP. |

---

## Verdict

**True Positive** — Confirmed malicious RDP brute force attack leading to a successful system compromise and subsequent unauthorized reconnaissance activity.

---

## Response Actions Taken

- Host device (Matthew) was successfully isolated from the network to prevent further actions or lateral movement.
- Completed playbook actions: Log Management, Determined Scope, Traffic Analysis, IP Reputation Check, and Enrichment & Context.
- Escalated for attacker eradication and review of network logs to ensure no internal communication occurred with other hosts during the compromise window.

---

## Recommendations

- **Block source IP** — Add 218.92.0.56 to firewall deny lists and relevant threat intelligence blocklists.
- **Disable external RDP** — Block RDP port (3389) exposure to the public internet. Require users to connect via a secure VPN before accessing internal endpoints via RDP.
- **Implement Account Lockout Policies** — Configure active directory/local security policies to lock accounts after a specified number of failed login attempts.
- **Enforce MFA** — Mandate Multi-Factor Authentication for all remote access and administrative logins.
- **Credential Reset** — Reset passwords for the compromised account(s) and ensure they meet complex length requirements.

---

## References

- VirusTotal analysis — Source IP 218.92.0.56
- ARIN Whois — Source IP 218.92.0.56
- MITRE ATT&CK: [T1110 — Brute Force](https://attack.mitre.org/techniques/T1110/)
- MITRE ATT&CK: [T1110.001 — Password Guessing](https://attack.mitre.org/techniques/T1110/001/)
