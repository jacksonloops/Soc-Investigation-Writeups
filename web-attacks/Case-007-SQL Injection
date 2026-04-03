# SOC Investigation Report — SOC127: SQL Injection Detected

---

**Information:**

| Field | Detail |
|---|---|
| Event Time | Mar 07, 2024, 12:51 PM |
| Severity | High |
| Rule | SOC127 — SQL Injection Detected |

---

## Executive Summary

SIEM triggered a high-severity SQL injection alert targeting an internal web server. Investigation determined that an external attacker launched multiple SQL injection attempts against the server. Based on available evidence — the SIEM alert indicating the requests were allowed and the absence of EDR coverage on the target host — the attack is presumed successful. The incident was escalated to Tier 2 for further containment and response.

---

## Alert Details

| Field | Detail |
|---|---|
| Rule Triggered | SOC127 — SQL Injection Detected |
| Log Source | Network logs |
| Source Address | 118.194.247.28 (External attacker) |
| Destination Address | 172.16.20.12 (Internal web server) |
| Observation | Multiple SQL injection attempts observed in request payloads |
| Status | Closed — Escalated to Tier 2 |

---

## Investigation Timeline

| Timestamp (UTC) | Action |
|---|---|
| 2024-03-07 12:51 | Alert triggered — SOC127: SQL Injection Detected |
| 2024-03-07 12:52 | Initial triage — confirmed source IP is external, destination IP is an internal web server |
| 2024-03-07 12:54 | Ran source IP through ARIN and VirusTotal — Chinese-registered IP, flagged by 9/91 vendors on VirusTotal, consistent with malicious activity |
| 2024-03-07 12:58 | Attempted EDR correlation — EDR not available on target host; network logs do not confirm whether injection payloads were executed |
| 2024-03-07 13:00 | Checked email logs for any scheduled penetration test or vulnerability scan — none found |
| 2024-03-07 13:02 | With the SIEM alert indicating requests were allowed and no additional telemetry to confirm or deny execution, the attack is presumed successful |
| 2024-03-07 13:03 | Escalated to Tier 2 for containment and further investigation |

---

## Findings

An external attacker operating from a Chinese-registered IP address (118.194.247.28) conducted multiple SQL injection attempts against an internal web server (172.16.20.12). The source IP is flagged as malicious by multiple threat intelligence vendors. Due to the lack of EDR coverage on the target host and the SIEM indicating requests were allowed, the attack is presumed successful. No evidence of a planned penetration test was found.

---

## MITRE ATT&CK Mapping

| Technique | ID | Description |
|---|---|---|
| Exploit Public-Facing Application | T1190 | Attacker exploited a public-facing web application to inject SQL commands targeting the backend database |

---

## Verdict

**True Positive** — Confirmed malicious SQL injection activity from an external source. Attack presumed successful based on available evidence.

---

## Response Actions Taken

- EDR not available on the target web server — remote containment not possible
- Escalated to Tier 2 for hands-on containment, forensic analysis, and remediation

---

## Recommendations

- **Block source IP** — Add 118.194.247.28 to firewall deny lists and relevant threat intelligence blocklists
- **Code remediation** — Conduct a code review of the web application to implement parameterized queries, input validation, and prepared statements to prevent SQL injection
- **Deploy a WAF** — Place a web application firewall in front of the server to detect and block injection attempts at the network layer
- **Extend EDR coverage** — Deploy endpoint detection and response tooling to the web server for improved visibility and future containment capability
- **Update detection signatures** — Add the observed injection patterns to IDS/IPS signature sets

---

## References

- VirusTotal analysis — Source IP 118.194.247.28
- Network logs — Internal web server 172.16.20.12
- MITRE ATT&CK: [T1190 — Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)
