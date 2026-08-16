# DVWA Web Application Security Assessment

Hands-on OWASP Top 10 vulnerability assessment performed against **DVWA (Damn Vulnerable Web Application)**, with **Metasploitable2** as a supporting lab target. Completed as part of a cybersecurity internship (ApexPlanet Software Pvt. Ltd. — Task 3, Days 25–36).

> ⚠️ **Disclaimer:** All testing was performed exclusively against intentionally vulnerable, locally hosted lab applications (DVWA / Metasploitable2) in an isolated environment. Nothing in this repository targets or was tested against production or third-party systems. This repo is for educational / portfolio purposes only.

---

## 👤 About

| | |
|---|---|
| **Name** | Mantu Kumar |
| **Role** | Cybersecurity Intern |
| **Program** | ApexPlanet Software Pvt. Ltd. — 60 Days Internship |
| **Task** | Task 3 — Web Application Security (Days 25–36) |
| **Assessment Dates** | 11 Aug 2026 – 15 Aug 2026 |

---

## 🎯 Objective

Identify and practically exploit OWASP Top 10 vulnerability classes in a controlled lab environment, and document each with:
- Reproducible steps
- Payloads used
- Request/response evidence (screenshots)
- Real-world impact
- Remediation guidance

---

## 🧪 Environment

| Component | Details |
|---|---|
| Target application | DVWA v1.10 (`http://127.0.0.1/DVWA/`) |
| Supporting target | Metasploitable2 (`192.168.225.130`) |
| Attacker OS | Kali Linux |
| Proxy/Testing tool | Burp Suite Professional v2026.3.3 |
| Browser | Firefox + FoxyProxy |
| DVWA security levels tested | Low, Medium |

---

## 🐞 Findings Summary

| # | Vulnerability | OWASP Top 10 (2021) | Severity | Status |
|---|---|---|---|---|
| 1 | [SQL Injection (Classic / UNION-based)](findings/01-sql-injection.md) | A03:2021 – Injection | 🔴 Critical | Confirmed |
| 2 | [SQL Injection (Blind, Time-Based)](findings/02-blind-sql-injection.md) | A03:2021 – Injection | 🟠 High | Confirmed |
| 3 | [OS Command Injection](findings/03-command-injection.md) | A03:2021 – Injection | 🔴 Critical | Confirmed |
| 4 | [Cross-Site Request Forgery (CSRF)](findings/04-csrf.md) | A01:2021 – Broken Access Control | 🟠 High | Confirmed |
| 5 | [Insecure CAPTCHA Implementation](findings/05-insecure-captcha.md) | A07:2021 – Auth Failures | 🟡 Medium | Confirmed |
| 6 | [Reflected Cross-Site Scripting (XSS)](findings/06-reflected-xss.md) | A03:2021 – Injection | 🟡 Medium | Confirmed |
| 7 | [Weak Authentication / Brute-Forceable Login](findings/07-brute-force.md) | A07:2021 – Auth Failures | 🟠 High | Confirmed |

Full detail on each finding — payloads, evidence, impact, and fixes — is in [`/findings`](findings/).

---

## 📚 OWASP Top 10 Reference

A full write-up of the OWASP Top 10 (2021) categories, what they mean, and how they map to this project's findings is in:

➡️ **[docs/OWASP_Top_10_2021.md](docs/OWASP_Top_10_2021.md)**

---

## 📁 Repository Structure

```
dvwa-security-assessment/
├── README.md                        # You are here
├── LICENSE
├── docs/
│   └── OWASP_Top_10_2021.md         # OWASP Top 10 (2021) deep-dive
├── findings/
│   ├── 01-sql-injection.md
│   ├── 02-blind-sql-injection.md
│   ├── 03-command-injection.md
│   ├── 04-csrf.md
│   ├── 05-insecure-captcha.md
│   ├── 06-reflected-xss.md
│   └── 07-brute-force.md
├── screenshots/
│   ├── 00-environment/
│   ├── 01-sql-injection/
│   ├── 02-blind-sqli/
│   ├── 03-command-injection/
│   ├── 04-csrf/
│   ├── 05-insecure-captcha/
│   ├── 06-reflected-xss/
│   └── 07-brute-force/
└── reports/
    └── DVWA_Security_Assessment_Report.docx   # Formal Word report
```

---

## 🛠️ Tools Used

- **Burp Suite Professional** — Proxy, Repeater, Intruder, CSRF PoC generator
- **Kali Linux** — attack platform
- **DVWA** — intentionally vulnerable target application
- **Metasploitable2** — supporting vulnerable lab VM
- **Hydra** — reference material reviewed for credential brute-forcing

---

## ✅ Key Takeaways

- Nearly every finding traces back to **untrusted input reaching a sensitive sink** (SQL engine, OS shell, HTML renderer) without validation/encoding.
- State-changing actions (password change, login) failed because **server-side verification was missing or bypassable** (no CSRF token, CAPTCHA state not tracked server-side, no login throttling).
- Fixing the same handful of root causes — parameterized queries, output encoding, anti-CSRF tokens, server-side state validation, and rate limiting — resolves the majority of the findings at once.

---

## 📄 License

This project is licensed under the [MIT License](LICENSE) — feel free to reference the methodology and write-up format for your own learning.

---

## 🙋 Contact

**Mantu Kumar** — Cybersecurity Intern
Feel free to connect for feedback or questions about this assessment.
