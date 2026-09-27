# 2. Threat Model & Risk Assessment (STRIDE)

*Report section: Threat Modelling, Risk Assessment and Threat-to-Control Mapping (LO2)*

This threat model is built against the architecture in `docs/architecture.svg`. Threats are
specific to NodeGoat's actual code, not generic template entries. Each is rated on a 5×5
likelihood × impact matrix and mapped to a concrete control and its location.

## 2.1 Risk matrix (5×5)

Score = Likelihood × Impact. **Low** 1–6, **Medium** 8–12, **High** 15–25.

| Likelihood \ Impact | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| **5** | 5 | 10 | 15 | 20 | 25 |
| **4** | 4 | 8 | 12 | 16 | 20 |
| **3** | 3 | 6 | 9 | 12 | 15 |
| **2** | 2 | 4 | 6 | 8 | 10 |
| **1** | 1 | 2 | 3 | 4 | 5 |

## 2.2 Identified threats

### T1 — Server-side JavaScript injection via `eval()` (STRIDE: Tampering, Elevation of Privilege)
- **Where:** `app/routes/contributions.js:32-34` — user-supplied `preTax`, `afterTax`,
  `roth` fields are passed straight into `eval()`.
- **Attack scenario:** an authenticated user submits a JavaScript expression instead of a
  number in the contributions form; it executes in the Node.js process, allowing arbitrary
  server-side code execution.
- **Likelihood 5 / Impact 5 = 25 (High).** Trivially reachable through a normal form; full
  server compromise.

### T2 — NoSQL injection via MongoDB `$where` (STRIDE: Information Disclosure, Tampering)
- **Where:** `app/data/allocations-dao.js:78` — the `threshold` value is concatenated into a
  `$where` clause as a raw string.
- **Attack scenario:** a crafted `threshold` value breaks out of the intended comparison and
  causes the `$where` JavaScript predicate to return other users' allocation records.
- **Likelihood 4 / Impact 4 = 16 (High).** Requires an authenticated request but exposes
  cross-user financial data.

### T3 — Stored Cross-Site Scripting in the profile (STRIDE: Tampering, Spoofing)
- **Where:** `app/routes/profile.js:64` — `firstName` (and other profile fields) are stored
  without sanitisation and later rendered in `app/views/profile.html`.
- **Attack scenario:** a user saves a script payload in a profile field; it is stored and
  executed in the browser of anyone who views that profile, enabling session theft.
- **Likelihood 4 / Impact 4 = 16 (High).** Persistent and affects other users/admins.

### T4 — Server-Side Request Forgery in stock research (STRIDE: Information Disclosure)
- **Where:** `app/routes/research.js:15-16` — the outbound request URL is built directly from
  `req.query.url`.
- **Attack scenario:** the user supplies an internal address (e.g. cloud metadata or the
  Mongo host); the server fetches it and returns the response, exposing internal services.
- **Likelihood 3 / Impact 4 = 12 (Medium).**

### T5 — Open redirect on the learning-resources link (STRIDE: Spoofing)
- **Where:** `app/routes/index.js:72` — `res.redirect(req.query.url)` with no allow-list.
- **Attack scenario:** a phishing link using the trusted app domain redirects victims to an
  attacker site after they are already logged in.
- **Likelihood 3 / Impact 2 = 6 (Low).**

### T6 — Vulnerable/EOL base image and dependencies (STRIDE: Elevation of Privilege)
- **Where:** `Dockerfile` (`node:12-alpine`, end-of-life) and `package.json` dependencies.
- **Attack scenario:** known CVEs in the OS packages and npm dependencies are exploitable in
  the running container.
- **Likelihood 4 / Impact 4 = 16 (High).**

## 2.3 Threat-to-control mapping

| # | Threat | Score | Control | Where the control lives |
|---|--------|-------|---------|--------------------------|
| T1 | SSJS injection (`eval`) | 25 | Replace `eval` with safe numeric parsing / whitelist input | Fix in `app/routes/contributions.js`; caught by Semgrep gate |
| T2 | NoSQL injection | 16 | Remove `$where` string concat; use typed query operators | Fix in `app/data/allocations-dao.js`; caught by Semgrep gate |
| T3 | Stored XSS | 16 | Server-side input validation + output escaping in views | Fix in `app/routes/profile.js` + `profile.html` |
| T4 | SSRF | 12 | Allow-list of permitted research hosts | Fix in `app/routes/research.js` |
| T5 | Open redirect | 6 | Relative-path / allow-list redirect only | Fix in `app/routes/index.js`; caught by Semgrep gate |
| T6 | Vulnerable base image/deps | 16 | Trivy container gate (blocks build) + npm audit gate; upgrade base image | `.github/workflows/security-pipeline.yml` |

Secrets exposure (hardcoded `cookieSecret`/`cryptoKey` in `config/env/all.js`) is treated
separately under Secrets Management (`docs/SECRETS_MANAGEMENT.md`) and is caught by the
Gitleaks gate.
