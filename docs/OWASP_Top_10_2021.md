# OWASP Top 10 (2021) — Reference Guide

The **OWASP Top 10** is a standard awareness document published by the Open Worldwide Application Security Project (OWASP), representing the most critical security risks to web applications. It is updated periodically based on data from hundreds of organizations and thousands of applications, combined with a community survey of security practitioners. The most recent full release is the **2021 edition**.

This document summarizes each category, explains it in practical terms, and — where applicable — links it to a finding demonstrated in this repository.

---

## A01:2021 – Broken Access Control

**What it means:** Users can act outside their intended permissions — viewing or modifying other users' data, accessing admin functionality, or bypassing authorization checks entirely.

**Common examples:**
- Modifying a URL or API parameter to view another user's account (Insecure Direct Object Reference — IDOR).
- Elevating privileges by tampering with a role parameter or JWT claim.
- Missing function-level access control (e.g. an unauthenticated user reaching an admin endpoint).
- **CSRF** is commonly grouped under this category, since it lets an attacker perform actions the victim didn't authorize.

**In this project:** [Finding 4 — CSRF](../findings/04-csrf.md) demonstrates a state-changing action (password change) performed without proper authorization/verification of the request's origin.

**Prevention:**
- Deny by default; explicitly grant access per role.
- Enforce access control server-side on every request, not just in the UI.
- Use anti-CSRF tokens for state-changing requests.
- Log access-control failures and alert on repeated violations.

---

## A02:2021 – Cryptographic Failures

**What it means:** Sensitive data (passwords, credit cards, health records, session tokens) is exposed due to weak or missing cryptography — previously known as "Sensitive Data Exposure."

**Common examples:**
- Transmitting sensitive data in cleartext (no HTTPS/TLS).
- Using outdated or weak algorithms (MD5, SHA1, DES) to store passwords.
- Hard-coded encryption keys or predictable initialization vectors.
- Missing encryption at rest for sensitive database fields.

**Relevance to this project:** The credential dump obtained in [Finding 1](../findings/01-sql-injection.md) exposed passwords stored as **MD5 hashes** — a weak, fast, unsalted hashing algorithm that is trivially crackable via rainbow tables or GPU brute-force, illustrating this category in practice.

**Prevention:**
- Use TLS everywhere; disable weak cipher suites and old protocol versions.
- Store passwords with a slow, salted algorithm (bcrypt, scrypt, or Argon2) — never a fast general-purpose hash like MD5/SHA1.
- Encrypt sensitive data at rest; manage keys via a dedicated key-management service.

---

## A03:2021 – Injection

**What it means:** Untrusted data is sent to an interpreter (SQL engine, OS shell, LDAP, XPath, etc.) as part of a command or query, allowing the attacker to alter its intended behavior. This category also absorbed **Cross-Site Scripting (XSS)** in the 2021 update.

**Common examples:**
- SQL Injection — attacker-controlled input alters a database query.
- OS Command Injection — attacker input is passed to a system shell.
- Cross-Site Scripting (XSS) — attacker script is reflected/stored and executed in a victim's browser.

**In this project:** This is the most represented category —
- [Finding 1 — SQL Injection (Classic/UNION)](../findings/01-sql-injection.md)
- [Finding 2 — SQL Injection (Blind)](../findings/02-blind-sql-injection.md)
- [Finding 3 — OS Command Injection](../findings/03-command-injection.md)
- [Finding 6 — Reflected XSS](../findings/06-reflected-xss.md)

**Prevention:**
- Use parameterized queries / prepared statements — never concatenate user input into queries or commands.
- Use an ORM with safe query builders where possible.
- Validate input against an allow-list; encode output for the correct context (HTML, JS, SQL, shell).
- Apply a Content-Security-Policy (CSP) to reduce the impact of any XSS that slips through.

---

## A04:2021 – Insecure Design

**What it means:** A new 2021 category focused on flaws in the *design and architecture* of an application, not just implementation bugs. Even flawless code can't fix a fundamentally insecure design.

**Common examples:**
- Missing threat modeling during design.
- Business logic that assumes trust (e.g. no transaction limits, missing rate limiting by design).
- Lack of "secure by default" patterns across the application.

**Relevance to this project:** The [Insecure CAPTCHA](../findings/05-insecure-captcha.md) finding is arguably a design flaw as much as an implementation one — the workflow was designed around a client-visible `step` parameter instead of a server-tracked verification state, meaning no amount of "patching" the CAPTCHA widget itself would fix the underlying trust assumption.

**Prevention:**
- Threat-model features before building them.
- Use secure design patterns and reference architectures.
- Assume all client-supplied state is attacker-controlled.

---

## A05:2021 – Security Misconfiguration

**What it means:** Insecure default configurations, incomplete setups, open cloud storage, verbose error messages, or unnecessary features left enabled.

**Common examples:**
- Default credentials left unchanged.
- Directory listing enabled; stack traces returned to users.
- Unnecessary ports, services, or debug features exposed in production.
- Security headers (CSP, X-Frame-Options, HSTS) missing.

**Relevance to this project:** The verbose MySQL/MariaDB error messages returned during column enumeration (see [Finding 1](../findings/01-sql-injection.md)) are a textbook example — production systems should never return raw database errors to the client.

**Prevention:**
- Harden and automate server configuration (infrastructure as code).
- Disable verbose error output in production; log details server-side only.
- Regularly review and remove unused features, frameworks, and default accounts.
- Apply standard security headers.

---

## A06:2021 – Vulnerable and Outdated Components

**What it means:** Using libraries, frameworks, or components with known vulnerabilities, or components that are no longer maintained.

**Common examples:**
- Running an outdated CMS, framework, or library with public CVEs.
- Not tracking component versions or subscribing to vulnerability feeds.
- Using components with unnecessary features, unused dependencies, or unpatched vulnerabilities.

**Relevance to this project:** DVWA and Metasploitable2 are themselves deliberately outdated/vulnerable-by-design applications — every module in this repo is, in a sense, an illustration of this category as well as the specific vulnerability class being tested.

**Prevention:**
- Maintain a Software Bill of Materials (SBOM); inventory every component and its version.
- Continuously monitor for CVEs (e.g. `npm audit`, `pip-audit`, Dependabot, Snyk).
- Patch or upgrade promptly; remove unused dependencies.

---

## A07:2021 – Identification and Authentication Failures

**What it means:** Weaknesses in how an application confirms a user's identity, authenticates sessions, and manages credentials — previously called "Broken Authentication."

**Common examples:**
- No rate limiting or lockout on login, enabling brute-force/credential-stuffing.
- Weak password policies; allowing default or common passwords.
- Session IDs exposed in URLs, or not rotated after login.
- Missing multi-factor authentication for sensitive accounts.

**In this project:**
- [Finding 5 — Insecure CAPTCHA](../findings/05-insecure-captcha.md) — the control meant to slow down automated abuse was bypassable.
- [Finding 7 — Brute Force / Weak Authentication](../findings/07-brute-force.md) — the login form had no lockout or throttling, allowing password guessing via Burp Intruder.

**Prevention:**
- Implement account lockout / exponential back-off after failed attempts.
- Require MFA, especially for privileged accounts.
- Enforce strong password policies; check against breached-password lists.
- Rotate session identifiers after login; use secure, HttpOnly, SameSite cookies.

---

## A08:2021 – Software and Data Integrity Failures

**What it means:** Code and infrastructure that doesn't verify integrity — e.g. relying on plugins, libraries, or CI/CD pipelines from untrusted sources without verification. New in 2021; includes insecure deserialization (folded in from the 2017 list).

**Common examples:**
- Auto-updating software from an untrusted source without signature verification.
- Insecure deserialization of untrusted data, leading to remote code execution.
- CI/CD pipelines without integrity checks, allowing malicious code injection.

**Prevention:**
- Verify digital signatures/checksums for software and dependencies.
- Ensure CI/CD pipelines have proper access control, segregation, and review.
- Avoid deserializing data from untrusted sources; use integrity checks (e.g. HMAC) when unavoidable.

---

## A09:2021 – Security Logging and Monitoring Failures

**What it means:** Insufficient logging, monitoring, and alerting means breaches go undetected for longer, giving attackers more time to escalate and exfiltrate data.

**Common examples:**
- Login attempts, access-control failures, and server-side input validation failures not logged.
- Logs only stored locally, tamperable by an attacker who gains access.
- No alerting pipeline, so anomalies are never reviewed in real time.

**Relevance to this project:** None of the attacks demonstrated in this repo (SQLi, brute force, CSRF, etc.) would have triggered any alert on a typical unmonitored setup like DVWA — reinforcing why logging/alerting on repeated failed logins, injection attempts, and unusual response-time patterns matters in production.

**Prevention:**
- Log authentication, access-control, and input-validation failures with enough context to investigate.
- Ensure logs are protected from tampering and retained appropriately.
- Establish monitoring and alerting for suspicious patterns (e.g. many failed logins, anomalous response times consistent with blind SQLi).

---

## A10:2021 – Server-Side Request Forgery (SSRF)

**What it means:** An application fetches a remote resource (URL) without validating the user-supplied destination, allowing an attacker to make the server issue requests to unintended locations — including internal-only services.

**Common examples:**
- A "fetch image from URL" or webhook feature that accepts any URL, including `http://169.254.169.254/` (cloud metadata endpoints) or internal-only hosts.
- Bypassing firewall/network segmentation because the request originates from the trusted server itself.

**Prevention:**
- Validate and allow-list destination hosts/IP ranges for any server-initiated request.
- Disable unnecessary URL schemas (`file://`, `gopher://`, etc.).
- Apply network-level segmentation so the application server cannot reach internal-only services it doesn't need.

---

## Summary Table

| # | Category | Demonstrated in this repo? |
|---|---|---|
| A01 | Broken Access Control | ✅ Finding 4 (CSRF) |
| A02 | Cryptographic Failures | ⚠️ Observed (MD5 password hashes in Finding 1 dump) |
| A03 | Injection | ✅ Findings 1, 2, 3, 6 |
| A04 | Insecure Design | ⚠️ Observed (Finding 5 design flaw) |
| A05 | Security Misconfiguration | ⚠️ Observed (verbose SQL errors, Finding 1) |
| A06 | Vulnerable & Outdated Components | ⚠️ Inherent to lab targets |
| A07 | Identification & Authentication Failures | ✅ Findings 5, 7 |
| A08 | Software & Data Integrity Failures | ➖ Not tested in this lab |
| A09 | Security Logging & Monitoring Failures | ➖ Not tested in this lab (no monitoring stack in scope) |
| A10 | Server-Side Request Forgery (SSRF) | ➖ Not tested in this lab |

---

## References

- OWASP Top 10 (2021): https://owasp.org/Top10/
- OWASP Testing Guide: https://owasp.org/www-project-web-security-testing-guide/
- OWASP Cheat Sheet Series: https://cheatsheetseries.owasp.org/
- DVWA GitHub: https://github.com/digininja/DVWA
