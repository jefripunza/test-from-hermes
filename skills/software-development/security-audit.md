---
name: security-audit
description: "Security auditor: OWASP Top 10, OWASP API Top 10. Detect SQLi, XSS, CSRF, SSRF, RCE, auth flaws, JWT vulns, data exposure. Security is mandatory."
metadata:
  hermes:
    tags: [security, owasp, audit, penetration-testing, vulnerability]
---

# Security Audit — Application Security Expert

Follow OWASP Top 10 + OWASP API Top 10. **Security is mandatory.**

## OWASP Top 10 (Detection)

1. **Broken Access Control** — IDOR, missing permission checks, privilege escalation
2. **Cryptographic Failures** — weak TLS, hardcoded secrets, weak hashing
3. **Injection** — SQLi, NoSQLi, Command Injection, LDAP Injection
4. **Insecure Design** — missing rate limits, trust boundaries wrong
5. **Security Misconfiguration** — default creds, debug enabled, CORS misconfig
6. **Vulnerable Components** — outdated deps, known CVEs, unpatched libs
7. **Authentication Failures** — weak passwords, no MFA, session fixation
8. **Integrity Failures** — unsigned updates, CSRF, deserialization attacks
9. **Logging Failures** — no audit trail, sensitive data in logs
10. **SSRF** — server-side request forgery, open redirects

## OWASP API Top 10 (Additional)

- **API1** — Broken Object Level Authorization
- **API2** — Broken Authentication
- **API3** — Broken Object Property Level Authorization
- **API4** — Unrestricted Resource Consumption
- **API5** — Broken Function Level Authorization
- **API6** — Unrestricted Access to Sensitive Business Flows
- **API7** — Server Side Request Forgery
- **API8** — Security Misconfiguration
- **API9** — Improper Inventory Management
- **API10** — Unsafe Consumption of APIs

## Detect

- SQL Injection
- XSS (Reflected, Stored, DOM-based)
- CSRF
- SSRF
- RCE (Remote Code Execution)
- Broken Authentication (JWT alg none, weak secrets)
- Broken Authorization (IDOR, privilege escalation)
- JWT Vulnerabilities (alg confusion, missing expiry, weak secret)
- Sensitive Data Exposure (PII, tokens in logs, secrets in source)
- Insecure Direct Object References (IDs in URLs without auth)
- Open Redirect
- Path Traversal
- Mass Assignment
- Race Conditions (TOCTOU)
- Dependency CVEs

## Requirements

- Input validation — server-side, not just client-side
- Output sanitization — escape HTML, JSON encoding
- Least privilege — minimal permissions, scoped tokens
- Secure defaults — opt-in to less secure features
- No secrets in source code — use env vars / secrets manager
- Rate limiting — prevent brute force
- HTTPS enforced — HSTS headers
- CSP headers — Content Security Policy

## Review Format

```text
Severity: Critical / High / Medium / Low
Issue: What vulnerability
Impact: What attacker can do
Recommendation: How to fix
Fix: Code/config change
```
