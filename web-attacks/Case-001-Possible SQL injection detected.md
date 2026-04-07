# SOC Investigation Report — Possible SQL Injection Detected

---

**Information:**

| Field | Detail |
|---|---|
| Event Time | \[From SIEM alert\] |
| Severity | Medium |
| Rule | Possible SQL Injection Detected — URL Contains `OR 1 = 1` |

---

## Executive Summary

A detection rule flagged an inbound HTTPS request containing SQL injection syntax targeting an internal web server. Investigation confirmed an external attacker operating from a DigitalOcean-hosted IP address attempted multiple SQL injection attacks against a search form. The decoded payloads contained classic injection patterns (`" OR 1 = 1 -- -`). All injection attempts were rejected by the server with HTTP 500 responses, meaning the attack was unsuccessful. No scheduled penetration test was found. The alert is confirmed as a true positive with no escalation required.

---

## Alert Details

| Field | Detail |
|---|---|
| Rule Triggered | Possible SQL Injection Detected — URL Contains `OR 1 = 1` |
| Log Source | Network / Web server logs |
| Source Address | 167.99.169.17 (External — DigitalOcean) |
| Source Port | 49575 |
| Destination Address | 172.16.17.18 (Internal web server) |
| Destination Port | 443 (HTTPS) |
| Traffic Direction | Internet → Company network |
| HTTP Response Codes | 500 (Server rejected all injection attempts) |
| Status | Closed — True Positive, Attack Failed |

---

## Investigation Timeline

| Timestamp (UTC) | Action |
|---|---|
| T+0:00 | Alert triggered — Possible SQL Injection Detected; requested URL contains `OR 1 = 1` |
| T+0:02 | Initial triage — reviewed alert indicators; confirmed the source IP is external and the destination is an internal web server on port 443 (HTTPS) |
| T+0:04 | Researched the source IP (167.99.169.17) via ARIN WHOIS/RDAP — IP is registered to DigitalOcean LLC; not a known company asset or authorized testing source |
| T+0:06 | Confirmed the destination (172.16.17.18) is an internal web server hosted within the company network |
| T+0:08 | Decoded the HTTPS request payload — revealed the injection attempt: `https://172.16.17.18/search/?q=" OR 1 = 1 -- -` |
| T+0:09 | Analyzed the payload — attacker injected a tautological condition (`OR 1 = 1`) into a search form parameter and used comment syntax (`-- -`) to truncate the remainder of the SQL query |
| T+0:11 | Checked email logs and internal communications for any scheduled penetration tests or vulnerability scans — none found |
| T+0:13 | Reviewed server response logs — all SQL injection attempts returned HTTP 500 status codes, indicating the server rejected the malicious requests; attack was unsuccessful |
| T+0:15 | Closed alert — confirmed true positive, no escalation required |

---

## Findings

An external attacker operating from a DigitalOcean-hosted IP address (167.99.169.17) conducted SQL injection attempts against an internal web server (172.16.17.18) over HTTPS. The decoded request payload reveals a classic SQL injection technique targeting a search form:

```
https://172.16.17.18/search/?q=" OR 1 = 1 -- -
```

The attacker injected a tautological condition (`OR 1 = 1`) designed to force a query to return all records, combined with SQL comment syntax (`-- -`) to neutralize the remainder of the original query. This is a standard technique for bypassing authentication or extracting data from vulnerable applications.

The source IP is not affiliated with the company network, and no scheduled penetration test or vulnerability scan was found in email logs or internal records. All injection attempts resulted in HTTP 500 server responses, confirming the server rejected the malicious input and the attack was unsuccessful.

---

## MITRE ATT&CK Mapping

| Technique | ID | Description |
|---|---|---|
| Exploit Public-Facing Application | T1190 | Attacker attempted to exploit a public-facing web application by injecting SQL commands into a search form parameter |

---

## Verdict

**True Positive** — Confirmed SQL injection attempt from an external source. Attack was unsuccessful — all injection payloads were rejected by the server (HTTP 500). No data compromise occurred.

---

## Response Actions Taken

- Confirmed all injection attempts were rejected by the server (HTTP 500 responses)
- Verified no scheduled penetration test or authorized scan was in progress
- No escalation required — attack was unsuccessful and no data was exposed

---

## Recommendations

- **Block source IP** — Add 167.99.169.17 to firewall deny lists and threat intelligence blocklists
- **Web application hardening** — While the server rejected the injection attempts, conduct a code review of the search form to ensure parameterized queries, input validation, and prepared statements are implemented; HTTP 500 responses may indicate the application crashed rather than gracefully handling the malicious input
- **Deploy a WAF** — Place a web application firewall in front of the server to detect and block SQL injection patterns at the network layer before they reach the application
- **Monitor for follow-up attacks** — Watch for additional reconnaissance or exploitation attempts from the same source IP or other DigitalOcean ranges targeting the same web server
- **Investigate the 500 response** — A server error (500) rather than a clean rejection (400/403) may indicate the application is vulnerable but failed to execute the payload due to a syntax or configuration issue; this should be reviewed by the development team

---

## References

- ARIN WHOIS/RDAP — Source IP 167.99.169.17 (DigitalOcean LLC)
- Web server access logs — 172.16.17.18 (HTTP 500 responses)
- Email logs — No scheduled penetration test found
- MITRE ATT&CK: [T1190 — Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)
