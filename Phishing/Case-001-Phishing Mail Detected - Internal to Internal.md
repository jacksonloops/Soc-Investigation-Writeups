# SOC Investigation Report — Phishing Mail Detected: Internal to Internal

---

**Information:**

| Field | Detail |
|---|---|
| Event Time | Feb 07, 2021, 04:24 AM |
| Severity | Medium |
| Rule | Phishing Mail Detected — Internal to Internal |

---

## Executive Summary

A phishing detection rule triggered on an internal-to-internal email between two employees. Investigation determined the alert was a false positive. The source and destination mailboxes showed no indicators of compromise, the email content contained no malicious elements or suspicious links, and the SMTP relay was confirmed as a legitimate internal Exchange server. No escalation or containment actions were required.

---

## Alert Details

| Field | Detail |
|---|---|
| Rule Triggered | Phishing Mail Detected — Internal to Internal |
| Log Source | Email gateway / SIEM |
| Source Email | john@letsdefend.io |
| Destination Email | susie@letsdefend.io |
| SMTP Address | 172.16.20.3 (Internal Exchange server) |
| Attachment | None |
| Suspicious Content | No |
| Status | Closed — False Positive |

---

## Investigation Timeline

| Timestamp (UTC) | Action |
|---|---|
| 2021-02-07 04:24 | Alert triggered — Phishing Mail Detected: Internal to Internal |
| 2021-02-07 04:26 | Initial triage — reviewed alert details; noted both sender and recipient are internal accounts, no attachments present |
| 2021-02-07 04:28 | Examined the email body and headers — no malicious links, suspicious language, or social engineering indicators found |
| 2021-02-07 04:31 | Investigated the source mailbox (john@letsdefend.io) — no signs of account compromise, no abnormal sending patterns or mail rule modifications |
| 2021-02-07 04:33 | Investigated the destination mailbox (susie@letsdefend.io) — no indicators of compromise or unusual activity |
| 2021-02-07 04:35 | Verified SMTP address 172.16.20.3 — confirmed as an internal Exchange server; no malicious activity or unauthorized relay behavior detected |
| 2021-02-07 04:37 | Assessed all findings — no evidence of phishing, compromise, or malicious intent identified |
| 2021-02-07 04:39 | Closed alert as false positive — no escalation required |

---

## Findings

The phishing detection rule flagged an internal-to-internal email sent from john@letsdefend.io to susie@letsdefend.io. A thorough review of the email content revealed no malicious URLs, attachments, or social engineering language. Investigation of both the source and destination mailboxes uncovered no signs of account compromise. The SMTP relay address (172.16.20.3) was confirmed as a legitimate internal Exchange server with no indicators of unauthorized use or malicious activity. The alert appears to have been triggered by a heuristic match rather than actual malicious behavior.

---

## MITRE ATT&CK Mapping

| Technique | ID | Relevance |
|---|---|---|
| Phishing: Spearphishing via Service | T1566.003 | Initially suspected — internal email flagged as potential phishing; ruled out after investigation |

---

## Verdict

**False Positive** — No malicious activity detected. The email, both mailboxes, and the SMTP server were all confirmed legitimate. Alert triggered by heuristic detection, not actual phishing.

---

## Response Actions Taken

- Reviewed email content, headers, and metadata — no malicious indicators
- Investigated source and destination mailboxes — no compromise detected
- Verified SMTP server legitimacy — confirmed internal Exchange server
- No containment or escalation required

---

## Recommendations

- **Tune detection rule** — Review the phishing detection heuristic that triggered this alert to reduce false positives on routine internal-to-internal email traffic
- **Baseline internal email patterns** — Establish known-good communication baselines between internal accounts to improve alert fidelity
- **Monitor for recurrence** — If similar false positives persist for this rule, consider adjusting threshold or adding exceptions for verified internal Exchange servers

---

## References

- Email gateway logs — john@letsdefend.io → susie@letsdefend.io
- SMTP server verification — 172.16.20.3 (Internal Exchange)
- MITRE ATT&CK: [T1566.003 — Phishing: Spearphishing via Service](https://attack.mitre.org/techniques/T1566/003/)
