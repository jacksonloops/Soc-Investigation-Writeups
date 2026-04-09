# SOC Investigation Report — JavaScript Code Detected in Requested URL

---

**Information:**

| Field | Detail |
|---|---|
| Event Time | (See Investigation Timeline) |
| Severity | Medium |
| Rule | JavaScript Code Detected in Requested URL |

---

## Executive Summary

A detection rule flagged inbound HTTPS requests containing JavaScript code targeting an internal web server's search form. Investigation confirmed an external attacker operating from a China Unicom IP address attempted multiple cross-site scripting (XSS) attacks by injecting JavaScript payloads into URL parameters. All attempts were met with HTTP 302 redirect responses, indicating the attack was unsuccessful. No scheduled penetration test was found. The alert is confirmed as a true positive with no escalation required.

---

## Alert Details

| Field | Detail |
|---|---|
| Rule Triggered | JavaScript Code Detected in Requested URL |
| Log Source | Network / Web server logs |
| Source Address | 112.85.42.13 (External — China Unicom, Jiangsu Province) |
| Destination Address | 172.16.17.17 (Internal web server) |
| Protocol | HTTPS |
| Observed Payload | `<script>javascript:alert(1)</script>` injected via search parameter |
| HTTP Response Codes | 302 (Redirect — all attempts failed) |
| Status | Closed — True Positive, Attack Failed |

---

## Investigation Timeline

| Timestamp (UTC) | Action |
|---|---|
| T+0:00 | Alert triggered — JavaScript Code Detected in Requested URL on internal web server 172.16.17.17 |
| T+0:02 | Initial triage — reviewed the flagged URL: `https://172.16.17.17/search/?q=<script>javascript:alert(1)</script>` — confirmed the request contains a JavaScript injection payload in the search parameter |
| T+0:04 | Researched the source IP (112.85.42.13) — not a company endpoint; ARIN WHOIS/RDAP lookup identifies it as China Unicom Jiangsu Province Network, a Chinese telecommunications service provider |
| T+0:06 | Confirmed the destination (172.16.17.17) is an internal company web server |
| T+0:08 | Analyzed network logs for all requests from the source IP — identified multiple requests containing various JavaScript XSS payloads targeting the search form |
| T+0:10 | Reviewed server response codes — all XSS injection attempts returned HTTP 302 (redirect) responses, indicating the payloads were not executed and the attack failed |
| T+0:12 | Checked email logs and internal communications for any scheduled penetration tests or vulnerability scans — none found |
| T+0:14 | Assessed all findings — confirmed true positive; attack unsuccessful, no escalation required |

---

## Findings

An external actor operating from a China Unicom IP address (112.85.42.13) conducted multiple cross-site scripting (XSS) attacks against an internal web server (172.16.17.17) over HTTPS. The attacker injected JavaScript payloads into the search form's URL parameter, with the initial observed payload being:

```
https://172.16.17.17/search/?q=<script>javascript:alert(1)</script>
```

This is a classic reflected XSS probe — the injected `alert(1)` function is commonly used to test whether a web application will execute arbitrary JavaScript in a user's browser. Multiple variations of this payload were attempted across several requests.

All injection attempts received HTTP 302 redirect responses from the server, meaning the malicious payloads were not rendered or executed. The attack was unsuccessful. No scheduled penetration test or authorized vulnerability scan was found in email logs. The source IP belongs to a major Chinese telecommunications provider, meaning it is likely a shared or dynamically assigned address rather than a dedicated attacker infrastructure.

---

## MITRE ATT&CK Mapping

| Technique | ID | Description |
|---|---|---|
| Exploit Public-Facing Application | T1190 | Attacker attempted to exploit a public-facing web application by injecting JavaScript code into URL parameters |

---

## Verdict

**True Positive** — Confirmed XSS attack attempt from an external source. All injection payloads failed — the server responded with HTTP 302 redirects and no JavaScript was executed. No compromise occurred.

---

## Response Actions Taken

- Confirmed all XSS injection attempts were unsuccessful (HTTP 302 responses)
- Verified no scheduled penetration test or authorized scan was in progress
- Source IP not blocked — it belongs to China Unicom, a major telecommunications provider; blocking could impact legitimate users on shared infrastructure
- No escalation required — attack was unsuccessful

---

## Recommendations

- **Web application hardening** — While the server redirected the malicious requests, conduct a code review of the search form to ensure proper input sanitization, output encoding, and Content Security Policy (CSP) headers are implemented to prevent XSS
- **Implement a Content Security Policy** — Deploy CSP headers on the web server to restrict the execution of inline scripts, providing an additional layer of defense against XSS even if input validation is bypassed
- **Deploy a WAF** — Place a web application firewall in front of the server to detect and block XSS patterns at the network layer before they reach the application
- **Monitor for follow-up attacks** — Watch for additional exploitation attempts from the same source IP or similar payloads targeting other parameters on the web server
- **Review the 302 behavior** — Investigate why the server responds with a redirect (302) rather than a clean rejection (400/403); ensure the redirect does not reflect any user-supplied input in the redirect target, as this could introduce an open redirect or reflected XSS vulnerability

---

## References

- ARIN WHOIS/RDAP — Source IP 112.85.42.13 (China Unicom, Jiangsu Province)
- Web server access logs — 172.16.17.17 (HTTP 302 responses)
- Email logs — No scheduled penetration test found
- MITRE ATT&CK: [T1190 — Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)
