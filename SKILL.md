---
name: skill-security-layer
description: >-
  Audits security and configuration of fullstack web applications from code the user pastes in chat.
  Generates a technical checklist with status ✅/❌/⚠️ evaluating essential security controls.
  Activate when the user requests to review, audit, or verify their project, or when they paste code
  asking if it's properly configured or secure.
compatibility: Framework agnostic. Works with any stack (Node, Python, PHP, etc.)
metadata:
  author: user
  version: "1.1"
---

# Project Checker — Security and Configuration Audit

Analyzes pasted code and produces a structured report with ✅/❌/⚠️ status on 40 security controls for fullstack web applications.

---

## Workflow

### 1. Collect the code

If no code has been pasted, ask:

> "Paste the code you want me to review — server config, middleware, `.env.example`, API routes, or any relevant fragment. Paste additional blocks one at a time; tell me when you're done."

Stop collecting when the user says they're done, or after 5 blocks, and proceed to analysis. Do not make assumptions about the stack — infer it from the code.

### 2. Analyze

For each check, look for **explicit evidence**. Apply these statuses:
- ✅ Correctly implemented
- ❌ Absent, incomplete, or cannot be determined from provided code
- ⚠️ Present but with a concrete risk (explain the risk, not just the fact)

**Severity tiers** (used in the summary):
- **P0** — Exploitable in production; block deployment
- **P1** — Significant risk; fix before launch
- **P2** — Hardening / best practice

### 3. Output the report

Use this format exactly:

---

## 🔍 Security Audit — `[project name or "Your Project"]` · `[detected stack]`

> Checks: 40 | Analyzed files: N

[Render one table per category using the template below. Fill Status and Detail for each row.]

**Table format:**
| # | Check | P | Status | Detail |
|---|---|---|---|---|
| N | Description | P0/P1/P2 | ✅/❌/⚠️ | Specific finding or "Not visible in provided code" |

---

### 📊 Summary

```
✅ Implemented:  X/40
⚠️ At risk:      X/40
❌ Absent/Unknown: X/40
```

**P0 — Fix before deployment:**
- [#N] Brief finding → concrete fix action

**P1 — Fix before launch:**
- [#N] Brief finding → concrete fix action

**P2 — Recommended hardening:**
- Only list items with ❌ or ⚠️

---

## Check Registry

Each check: `ID · Description · Severity`

### 🌐 CORS / Security Headers
1. `Access-Control-Allow-Origin` restricted — no `*` in production · P0
2. HTTP methods explicitly allowlisted · P1
3. `X-Frame-Options`, `X-Content-Type-Options`, `HSTS`, `CSP` present · P1
4. CORS credentials handled correctly (`credentials: include` + `allowCredentials`) · P1

### 🚦 Rate Limiting
5. Rate limiting on all API routes · P1
6. Rate limiting on auth endpoints (login, register, password reset) · P0
7. Limits explicitly configured (time window + max requests) · P1
8. 429 response with `Retry-After` header · P2

### 🔐 Authentication / Tokens
9. JWT/tokens signed with strong secret — not hardcoded · P0
10. Token expiration defined (`expiresIn` / `exp`) · P0
11. Refresh token strategy implemented or absence justified · P1
12. Tokens stored securely — not `localStorage` for sensitive tokens · P1
13. Protected routes guarded by authentication middleware · P0
14. Passwords hashed (bcrypt, argon2) — never plaintext · P0

### ⚙️ Environment Variables / Configuration
15. Secrets in environment variables — not hardcoded in source · P0
16. `.env` in `.gitignore` · P0
17. `.env.example` present with dummy values only · P2
18. Env vars validated at startup (fail-fast if missing) · P1
19. `NODE_ENV` / equivalent distinction enforced between dev and prod · P1
20. No real secrets in git history (checked via grep patterns or documented process) · P0

### 🛡️ Input Validation / Sanitization
21. Explicit input validation on API routes (schema, type, range, regex) · P0
22. Data sanitized before persistence (escape, strip tags, trim) · P1
23. Oversized payloads rejected (`maxBodySize` / `content-length` limit) · P1

### 🧬 Injection
24. Parameterized queries / prepared statements used (no string concatenation with user input) · P0
25. ORM/ODM configured to prevent raw query injection · P0
26. No `eval`, `exec`, or dynamic code execution with user input · P0

### 🕸️ XSS & CSRF
27. Output encoding when rendering dynamic content (HTML, JS, URL, CSS contexts) · P0
28. CSRF tokens on state-changing mutations (`POST`, `PUT`, `DELETE`) · P1
29. `Content-Type` forced; CSP / `X-XSS-Protection` configured · P1

### 🖼️ File Uploads
30. MIME type and extension validated via whitelist · P1
31. Files stored outside web root, renamed with UUID, not executable · P1

### 🍪 Cookies & Sessions
32. `HttpOnly`, `Secure`, `SameSite=Strict/Lax` on session/token cookies · P0
33. Session rotation post-login; server-side invalidation on logout · P1

### 📜 Error Handling & Logging
34. Stack traces and internal details not exposed in 4xx/5xx responses · P0
35. Security events logged (failed login, access denied, rate-limit hit) · P1
36. Correlation/request IDs for cross-service traceability · P2

### 📦 Dependencies & Supply Chain
37. Dependencies audited (`npm audit`, `pip-audit`, `snyk`) — no critical CVEs · P1
38. Lockfile present and committed (`package-lock.json`, `poetry.lock`, etc.) · P1
39. Dependency integrity verified (checksums / `--frozen-lockfile` in CI) · P2

### 🔗 Open Redirect & Miscellaneous
40. Redirect destinations validated against an allowlist (no open redirect via `url`, `next`, `redirect` params) · P1

---

## Analysis Rules

- If a check **cannot be determined** from the provided code, mark ❌ with `"Not visible in provided code"` — never assume it's fine.
- Be specific: reference the actual function, variable, or line when possible.
- Never repeat the user's code in the report — use precise references only.
- Distinguish **explicit configuration** from **framework defaults**. If a framework handles something by default but it is not explicitly configured, use ⚠️ and note the assumption.
- Adapt recommendations to the detected stack: name the specific package or middleware (e.g., `express-rate-limit`, `slowapi`, `helmet`).
- If a seemingly risky configuration has a valid technical justification visible in context, mark ⚠️ and explain the residual risk.

## Out of Scope

Do not cover: performance, accessibility, SEO, business logic correctness, or infrastructure/cloud configuration unless the user explicitly requests it.

## Behavior After the Report

Offer:
> "Want me to go deeper on any point or show a ready-to-paste fix for any ❌?"

If the user accepts, provide **only the affected fragment** — not the full file. Include:
- The technical risk in one sentence.
- The corrected code, ready to copy-paste.
- The recommended library/middleware for the detected stack.

Do not offer to rewrite the entire codebase unless explicitly asked.
