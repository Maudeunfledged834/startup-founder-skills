---
name: security-review
trigger: When the user needs a security assessment — threat modeling, vulnerability review, auth flow audit, dependency scanning, or says "is this secure", "review for vulnerabilities", "threat model", "security audit", "pen test prep".
related: [code-review, architecture-design, soc2-prep]
reads: [startup-context]
---

# Security Review

## When to Use

- The user wants a security audit of their application or specific feature
- They need a threat model or auth flow review before launching
- They have a dependency vulnerability alert and need remediation guidance
- They are handling sensitive data (PII, payment, health) and need verification

## Context Required

From `startup-context`: tech stack, deployment environment, compliance requirements, data types. Also ask:
- Scope of review (full app, feature, auth system, single PR)
- Data types handled (PII, payment, health, credentials)
- Deployment environment and compliance requirements (SOC 2, HIPAA, PCI-DSS, GDPR)

## Workflow

1. **Define scope** — Architecture, code, auth flows, dependencies, infrastructure, or all.
2. **STRIDE threat model** — Walk through each STRIDE category against the system.
3. **OWASP Top 10 audit** — Check each category with stack-specific tests.
4. **Auth flow review** — Trace auth/authz end to end: tokens, sessions, privilege escalation.
5. **Dependency scan** — Identify vulnerable deps, score by CVSS, assess exploitability.
6. **Infrastructure review** — Misconfigs, exposed ports, permissive IAM, missing encryption.
7. **Score and prioritize** — CVSS scores, categorize by severity and exploitability.
8. **Deliver report** — Structured findings with remediation and prioritized action plan.

## Output Format

```markdown
# Security Review: [Scope Description]

## Executive Summary
Overall risk posture (Critical / High / Medium / Low) and top findings.

## Threat Model (STRIDE)
| Threat | Category | Asset | Impact | Likelihood | Risk |

## Findings
### Critical / High / Medium / Low
- **[SEC-N] Title** — CVSS X.X — description, impact, remediation

## Auth Flow Assessment
## Dependency Vulnerabilities
| Package | Current | CVSS | Fix Version | Exploitable? |

## Remediation Roadmap
```

## Frameworks & Best Practices

### STRIDE Threat Modeling

Apply to every component and data flow:
- **Spoofing** — Can attackers forge tokens? API keys rotatable?
- **Tampering** — Requests modified in transit? Webhooks signed?
- **Repudiation** — Critical actions logged? Tamper-evident logs?
- **Information Disclosure** — Stack traces in errors? PII encrypted at rest?
- **Denial of Service** — Rate limits? Can one user exhaust resources?
- **Elevation of Privilege** — Regular users access admin? Server-side role checks?

### OWASP Top 10 Checks

1. **Injection** — Parameterize SQL/NoSQL; check OS commands, SSTI, LDAP
2. **Broken Auth** — argon2id/bcrypt, session timeout, rate limiting
3. **Data Exposure** — TLS 1.2+, PII encrypted at rest, HSTS
4. **XXE** — Disable DTD, prefer JSON
5. **Access Control** — Server-side authz, no IDOR, CORS whitelist
6. **Misconfig** — Debug off, default creds removed, security headers
7. **XSS** — Output encoding, CSP, HTTP-only cookies
8. **Deserialization** — Validate schema, prefer JSON
9. **Vulnerable Deps** — `npm audit`, `pip-audit`, `trivy`
10. **Logging** — Auth events, admin actions, alerts

### CVSS v3.1 Scoring

- **Critical (9.0-10.0):** RCE, auth bypass, full data breach
- **High (7.0-8.9):** Privilege escalation, significant data exposure, SSRF
- **Medium (4.0-6.9):** Stored XSS, CSRF, limited IDOR
- **Low (0.1-3.9):** Info disclosure, missing headers

### Auth Flow Checklist

- [ ] Passwords: argon2id or bcrypt (cost >= 10)
- [ ] JWT: 15-min access tokens, 7-day refresh tokens rotated on use
- [ ] Rate limiting (5 attempts / 15 min), sessions invalidated on password change
- [ ] OAuth state param, scoped API keys, MFA for admins, single-use reset tokens

### Security Scanning Tools

- **Node.js:** `semgrep` + `eslint-plugin-security` (SAST), `npm audit` / `snyk` (deps)
- **Python:** `bandit` + `semgrep` (SAST), `pip-audit` (deps)
- **Go:** `gosec` (SAST), `govulncheck` (deps)
- **All stacks:** `semgrep --config=auto` (SAST), `trivy fs .` / `trivy image` (deps + containers)

### Remediation Priority

1. Actively exploitable + critical data — immediately
2. Auth/authz bypass — 24 hours
3. Injection — 48 hours
4. Data exposure / critical CVEs — 1 week
5. Config hardening — 2 weeks
6. Defense-in-depth — next sprint

## Related Skills

- `code-review` — chain when findings require code-level fixes
- `architecture-design` — chain when findings reveal architectural flaws
- `soc2-prep` — chain when review is part of compliance preparation

## Examples

**Example prompt:** "Review the security of our user authentication system. We use JWT with Express."

**Good output snippet:**
```
# Security Review: JWT Authentication System

## Executive Summary
Risk posture: **High**. JWT secret hardcoded in source, tokens never expire.

## Findings
### Critical (CVSS 9.8)
- **[SEC-1] Hardcoded JWT secret** — auth/config.js:3 — Secret is
  `"supersecret123"`. Attacker can forge tokens for any user.
  **Fix:** Move to env var, generate with `openssl rand -base64 64`,
  rotate every 90 days. Consider RS256 with key pair.

### Critical (CVSS 9.1)
- **[SEC-2] Tokens never expire** — auth/jwt.js:12 — No `expiresIn`
  set. Leaked token grants permanent access.
  **Fix:** Set `expiresIn: '15m'`, add refresh tokens (7-day, rotated).
```

**Example prompt:** "Snyk alert about CVE in lodash. What should we do?"

**Good output snippet:**
```
## lodash (CVE-2021-23337, CVSS 7.2) — command injection via `_.template`
1. Search codebase for `_.template` with user input — if unused, low risk
2. Upgrade: `npm install lodash@4.17.21` (patch, no breaking changes)
3. Add `npm audit --audit-level=high` to CI going forward
```
