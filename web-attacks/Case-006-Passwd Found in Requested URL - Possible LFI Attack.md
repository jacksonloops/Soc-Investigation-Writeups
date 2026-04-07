# SOC Investigation Report — Incident: Passwd Found in Requested URL — Possible LFI Attack

---

**Information:**

| Field | Detail |
|---|---|
| Event Time | (See Investigation Timeline) |
| Severity | Medium |
| Rule | Passwd Found in Requested URL — Possible LFI Attack |

---

## Executive Summary

A detection rule flagged an inbound HTTPS request to an internal web server after identifying "passwd" in the requested URL, indicating a possible Local File Inclusion (LFI) attack. Investigation confirmed the source IP belongs to Tencent Cloud, a cloud service provider based in Beijing, with no affiliation to the organization. Log analysis revealed the attacker attempted a classic LFI traversal to access the server's password file. However, all attempts returned HTTP 500 response codes — the server rejected the requests and no data was exposed. Email logs were reviewed to rule out any authorized penetration testing — none was scheduled. The attack was unsuccessful, and no containment or escalation was required.

---

## Alert Details

| Field | Detail |
|---|---|
| Rule Triggered | Passwd Found in Requested URL — Possible LFI Attack |
| Log Source | Network / IDS |
| Source IP | 106.55.45.162 |
| Source Port | 49028 |
| Destination IP | 172.16.17.13 |
| Destination Port | 443 |
| Protocol | HTTPS |
| Traffic Direction | Internet → Company Network |
| Status | Closed — True Positive, Attack Unsuccessful |

---

## Investigation Timeline

| Step | Action |
|---|---|
| 1 | Alert triggered — "passwd" string detected in the URL of an inbound HTTPS request to 172.16.17.13 |
| 2 | Investigated source IP (106.55.45.162) via ARIN WHOIS/RDAP — identified as belonging to Tencent Cloud Computing (Beijing); no affiliation with the organization |
| 3 | Investigated destination IP (172.16.17.13) — confirmed as an internal company web server |
| 4 | Reviewed web server logs — identified requests containing classic LFI directory traversal patterns attempting to access the server's passwd file |
| 5 | Analyzed HTTP response codes — all LFI attempts returned HTTP 500 (Internal Server Error); the server did not serve the requested file |
| 6 | Reviewed email logs to determine whether a penetration test or security assessment was scheduled — no authorized testing was found |
| 7 | Confirmed the attack was unsuccessful — no data was exposed and no further action required |

---

## Findings

An external attacker operating from a Tencent Cloud IP address (106.55.45.162) targeted an internal company web server (172.16.17.13) over HTTPS. The attacker attempted a Local File Inclusion (LFI) attack by embedding directory traversal sequences and a reference to the system's passwd file in the requested URL. Log analysis confirmed classic LFI request patterns consistent with an attempt to read sensitive server files. All requests returned HTTP 500 response codes, indicating the server encountered an error and did not serve the requested file — the attack was unsuccessful and no data was exposed. The source IP has no connection to the organization, and a review of email logs confirmed no penetration test or authorized security assessment was scheduled. The traffic flow was inbound from the public internet to the company network.

---

## MITRE ATT&CK Mapping

| Technique | ID | Description |
|---|---|---|
| Exploit Public-Facing Application | T1190 | Attacker attempted to exploit the web server via LFI directory traversal in the URL |
| File and Directory Discovery | T1083 | Attacker attempted to access the system passwd file to enumerate users and system information |

---

## Verdict

**True Positive** — Confirmed LFI attack attempt against an internal web server from an external source. The attacker attempted to read the server's passwd file via directory traversal. All requests returned HTTP 500 — the attack was **unsuccessful** and no data was exposed. No containment or escalation required.

---

## Response Actions Taken

- No containment required — all attack attempts failed with HTTP 500 responses
- No escalation required — the web server was not compromised and no data was exposed
- Alert closed as true positive, attack unsuccessful

---

## Recommendations

- **Block source IP** — Add 106.55.45.162 to firewall deny lists and threat intelligence blocklists
- **Web Application Firewall (WAF)** — Deploy or tune WAF rules to detect and block LFI patterns such as directory traversal sequences (`../`) and references to sensitive system files (e.g., `/etc/passwd`) in request URLs
- **Input validation hardening** — While the server rejected the requests, the HTTP 500 response suggests the application threw an unhandled error rather than explicitly blocking the malicious input; implement proper input sanitization to reject traversal patterns with an appropriate 400-series response
- **Network-wide sweep** — Search logs for additional connections from the source IP or related Tencent Cloud ranges to identify any other targeted systems
- **Monitor for follow-up activity** — The attacker may attempt alternative techniques against the same or other public-facing assets; monitor for further probing from the source IP or associated infrastructure

---

## References

- ARIN WHOIS/RDAP — Source IP 106.55.45.162 (Tencent Cloud Computing, Beijing)
- Web server access logs — Request analysis and HTTP response codes
- Email logs — Penetration test verification
- MITRE ATT&CK: [T1190 — Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)
- MITRE ATT&CK: [T1083 — File and Directory Discovery](https://attack.mitre.org/techniques/T1083/)
