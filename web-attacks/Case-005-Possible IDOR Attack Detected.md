# SOC Investigation Report — Incident: Possible IDOR Attack Detected

---

**Information:**

| Field | Detail |
|---|---|
| Event Time | (See Investigation Timeline) |
| Severity | High |
| Rule | Possible IDOR Attack Detected |

---

## Executive Summary

A detection rule flagged a series of consecutive HTTPS requests to the same page on an internal web server, indicating a possible Insecure Direct Object Reference (IDOR) attack. Investigation confirmed the source IP belongs to DigitalOcean, a cloud service provider, with no affiliation to the organization. Log analysis revealed the attacker made repeated requests manipulating object references to enumerate and retrieve other users' data. All requests returned HTTP 200 response codes, confirming the server responded with data for each attempt. Email logs were reviewed to rule out any authorized penetration testing — none was scheduled. The affected web server was contained and the incident was escalated to Tier 2 for full forensic analysis, data exposure assessment, and remediation.

---

## Alert Details

| Field | Detail |
|---|---|
| Rule Triggered | Possible IDOR Attack Detected |
| Log Source | Network / IDS |
| Source IP | 134.209.118.137 |
| Source Port | 49211 |
| Destination IP | 172.16.17.15 |
| Destination Port | 443 |
| Protocol | HTTPS |
| Traffic Direction | Internet → Company Network |
| Status | Closed — Contained and Escalated to Tier 2 |

---

## Investigation Timeline

| Step | Action |
|---|---|
| 1 | Alert triggered — consecutive requests to the same page detected from a single external source targeting 172.16.17.15 |
| 2 | Investigated source IP (134.209.118.137) via ARIN WHOIS/RDAP — identified as belonging to DigitalOcean LLC, a cloud service provider; no affiliation with the organization |
| 3 | Investigated destination IP (172.16.17.15) — confirmed as an internal company web server |
| 4 | Reviewed web server logs — identified multiple requests from the source IP systematically manipulating object references to access other users' data |
| 5 | Analyzed HTTP response codes — all IDOR attempts returned HTTP 200, confirming the server processed the requests and returned data |
| 6 | Reviewed email logs to determine whether a penetration test or security assessment was scheduled — no authorized testing was found |
| 7 | Contained the affected web server — full network isolation applied |
| 8 | Escalated the incident to Tier 2 / IR team for full forensic analysis, data exposure assessment, and remediation |

---

## Findings

An external attacker operating from a DigitalOcean-hosted IP address (134.209.118.137) targeted an internal company web server (172.16.17.15) over HTTPS. The attacker exploited an Insecure Direct Object Reference (IDOR) vulnerability by sending repeated requests to the same endpoint while manipulating object identifiers to access data belonging to other users. Log analysis confirmed multiple sequential requests consistent with user data enumeration, and all attempts returned HTTP 200 response codes — meaning the server responded with the requested data each time. The source IP has no connection to the organization, and a review of email logs confirmed no penetration test or authorized security assessment was scheduled. The traffic flow was inbound from the public internet to the company network. User data was likely exposed to the attacker.

---

## MITRE ATT&CK Mapping

| Technique | ID | Description |
|---|---|---|
| Exploit Public-Facing Application | T1190 | Attacker exploited an IDOR vulnerability in the company web server to access unauthorized data |
| Brute Force: Credential Stuffing | T1110.004 | Attacker systematically enumerated object references to retrieve other users' information (analogous enumeration technique) |
| Data from Information Repositories | T1213 | User data was retrieved from the web application through manipulated requests |

---

## Verdict

**True Positive** — Confirmed IDOR attack against an internal web server from an external source. The attacker successfully enumerated and retrieved other users' data through manipulated object references. All requests returned HTTP 200 responses, indicating data was exposed. No authorized testing was scheduled. The server is compromised and user data has potentially been exfiltrated.

---

## Response Actions Taken

- Contained the affected web server via EDR — full network isolation applied
- Escalated to Tier 2 / IR team for full forensic analysis, data exposure assessment, and remediation

---

## Recommendations

- **Block source IP** — Add 134.209.118.137 to firewall deny lists and threat intelligence blocklists
- **Patch the web application** — Implement proper authorization checks on all endpoints to ensure users can only access their own data; enforce server-side access control validation on every object reference
- **Data breach assessment** — Determine exactly which user records were accessed by correlating the HTTP 200 responses with the manipulated object identifiers in the logs; initiate data breach notification procedures if required
- **Web Application Firewall (WAF)** — Deploy or tune WAF rules to detect and rate-limit sequential requests with systematically incremented or manipulated parameters
- **Network-wide sweep** — Search logs for additional connections from the source IP or related DigitalOcean ranges to identify any other targeted systems
- **API and endpoint audit** — Review all public-facing API endpoints and web pages for similar IDOR vulnerabilities where object-level authorization is missing
- **Rate limiting** — Implement rate limiting on sensitive endpoints to slow down enumeration attempts

---

## References

- ARIN WHOIS/RDAP — Source IP 134.209.118.137 (DigitalOcean LLC)
- Web server access logs — Request analysis and HTTP response codes
- Email logs — Penetration test verification
- MITRE ATT&CK: [T1190 — Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)
- MITRE ATT&CK: [T1110.004 — Brute Force: Credential Stuffing](https://attack.mitre.org/techniques/T1110/004/)
- MITRE ATT&CK: [T1213 — Data from Information Repositories](https://attack.mitre.org/techniques/T1213/)
