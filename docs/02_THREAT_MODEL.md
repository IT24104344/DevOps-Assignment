# 2. Threat Model & Risk Assessment (STRIDE)

*Report section: Threat Modelling, Risk Assessment and Threat-to-Control Mapping (LO2). Owner: Member 2.*

**Method:** STRIDE per component and data flow, plus a 3×3 risk matrix
**System under test:** OWASP NodeGoat v1.3.0 (upstream commit `c5cb68a`), run locally with Docker Compose.

## 1. System overview

NodeGoat is an employee retirement-savings portal. Users log in, edit their profile (name, SSN, date of birth, bank details), set 401(k) contribution percentages, view stock allocations and post memos. An admin user manages benefit start dates for all employees.

| Component | Technology | Runs as | Exposure |
|---|---|---|---|
| `web` container | Node.js 12, Express 4, express-session, Swig templates | user `node` | Port 4000 published to the host |
| `mongo` container | MongoDB 4.4 | user `mongodb` | Port 27017 only on the Compose network |

Architecture diagram: [`architecture.png`](architecture.png) (editable source: `architecture.svg`).

### Assets
1. **PII and financial data:** SSN, DOB, address, bank account and routing numbers (`users` collection).
2. **Credentials and sessions:** passwords and the session cookie.
3. **Retirement data:** contribution percentages and stock allocations.
4. **Server integrity:** the Node.js process and container. Code execution here exposes everything above.

### Trust boundaries
- **TB1, Internet → web app:** every form field, query string and URL parameter is attacker-controlled.
- **TB2, web app → MongoDB:** the database has no authentication and trusts anything that reaches it on the Docker network.

## 2. STRIDE threats

| ID | STRIDE | Threat | Where in NodeGoat | Linked vulnerability (owner) |
|---|---|---|---|---|
| T1 | **E**levation of privilege | User input from the contributions form is passed to `eval()`, so an attacker can run their own code on the server inside the web container. | `app/routes/contributions.js` lines 32-34 (Semgrep ERROR `code-string-concat`) | V1 server-side JS injection (M3) |
| T2 | **T**ampering / **I**nfo disclosure | The `threshold` query parameter on the allocations page is concatenated into a MongoDB `$where` expression, so crafted input changes the query logic and can return other users' data or tie up the database. | `app/data/allocations-dao.js` `getByUserIdAndThreshold` | V2 NoSQL injection (M4) |
| T3 | **I**nfo disclosure / **S**poofing | Profile fields such as first name are stored and later rendered without HTML encoding (Swig `autoescape: false`). Script saved by one user runs in the browser of anyone who views it, including the admin on the Benefits page, and can act as that user (read their PII on screen, submit forms, redirect them). | `server.js` Swig defaults; `app/views/layout.html`, `benefits.html` | V3 stored XSS (M2) |
| T4 | **S**poofing | Passwords are stored and compared in plaintext, so anyone who reads the `users` collection (e.g. via T1/T2) can log in as any employee, including admin. | `app/data/user-dao.js` `addUser` / `validateLogin` | V4 broken authentication (M1) |
| T5 | **E**levation of privilege | The admin-only Benefits routes have no role check, and allocations are looked up by the `userId` in the URL rather than the session, so a normal user can reach admin functions and other users' records. | `app/routes/index.js` lines 55-56; `app/routes/allocations.js` | Access control (extra) |
| T6 | **I**nfo disclosure | Secrets are hardcoded in source (session signing secret, crypto key) and MongoDB has no auth, so leaking the repo or reaching the DB port exposes sessions and data. | `config/env/all.js`; `docker-compose.yml` | Secrets management (M1) |
| T7 | **D**enial of service | The bank-routing validation regex is prone to catastrophic backtracking, so one long crafted value can pin the CPU of the single Node.js process. | `app/routes/profile.js` `regexPattern` | Extra (report only) |
| T8 | **R**epudiation | There is no audit log of profile, contribution or benefit changes (only `console.log`), so a user can deny changing another employee's benefits and there is no evidence trail. | Whole app | Extra (report only) |
| T9 | **I**nfo disclosure | The Research page builds an outbound request URL from the `url` query parameter, so the server can be made to fetch internal addresses (SSRF). | `app/routes/research.js` lines 15-16 | Spare vulnerability |
| T10 | **S**poofing | `/learn` redirects to any `url` query value with no allow-list (open redirect), useful for phishing from a trusted domain. | `app/routes/index.js` line 72 (Semgrep `express-open-redirect`) | Spare vulnerability |
| T11 | **E**levation of privilege | The image uses end-of-life `node:12-alpine` and old npm packages with known CVEs. | `Dockerfile`, `package.json` | Trivy + npm audit gates (M4) |

## 3. Risk matrix (likelihood × impact, 1 = low, 3 = high)

| Likelihood ↓ / Impact → | **Low (1)** | **Medium (2)** | **High (3)** |
|---|---|---|---|
| **High (3)** | | T8 | **T1, T3, T5** |
| **Medium (2)** | | T7, T9 | **T2, T4, T6, T11** |
| **Low (1)** | | T10 | |

| ID | Likelihood | Impact | Score | Justification |
|---|---|---|---|---|
| T1 | 3 | 3 | **9 Critical** | Any logged-in user can reach the form and Semgrep flags it as ERROR; code execution compromises the container and, through it, the unauthenticated DB. |
| T3 | 3 | 3 | **9 Critical** | Needs only a normal account and a profile edit; the admin views every user's name on Benefits, so admin session theft is realistic. |
| T5 | 3 | 3 | **9 Critical** | Changing a URL or visiting a route needs no skill; exposes other employees' financial data and admin functions. |
| T2 | 2 | 3 | **6 High** | Needs knowledge of MongoDB `$where` syntax, but leaks every user's allocations and can stall the DB. |
| T4 | 2 | 3 | **6 High** | Needs a second flaw to read the DB first, but then gives full account takeover of every user. |
| T6 | 2 | 3 | **6 High** | Requires repo or network access; public repos are routinely scraped for secrets. |
| T7 | 2 | 2 | **4 Medium** | Easy to trigger, but only affects availability and recovers on restart. |
| T8 | 3 | 2 | **6 Medium** | Disputes are likely in a finance app, but the harm is to accountability rather than data. |
| T9 | 2 | 2 | **4 Medium** | Needs knowledge of internal hosts; inside Docker the reachable services are limited to MongoDB. |
| T10 | 1 | 2 | **2 Low** | Needs a victim to click a crafted link; only aids phishing. |
| T11 | 2 | 3 | **6 High** | Public CVEs exist for the EOL base image; exploitability depends on the specific package. |

## 4. Threat-to-control mapping

Fill in the "Evidence" column with commit/PR links once each fix is merged.

| ID | Control | Where the control lives | Pipeline gate that catches regressions | Evidence |
|---|---|---|---|---|
| T1 | Replace `eval()` with `parseInt()` and range validation | `app/routes/contributions.js` (branch `fix/v1-eval`) | Semgrep (fails on ERROR) | M3 PR |
| T2 | Parse `threshold` as an integer 0-99 and query with operators instead of `$where` strings | `app/data/allocations-dao.js` (branch `fix/v2-nosql`) | Semgrep | M4 PR |
| T3 | Swig `autoescape: true`; Content-Security-Policy `script-src 'self'` via helmet; session cookie `httpOnly: true` set explicitly; inline login script moved to `/js/cookie-check.js` so CSP does not break it | `server.js`, `app/views/memos.html`, `app/views/login.html`, `app/assets/js/cookie-check.js` (branch `m2/threat-model-xss`) | Semgrep, optional ZAP baseline | M2 PR |
| T4 | bcrypt hash on signup, `bcrypt.compareSync` on login, hashed seed users | `app/data/user-dao.js`, `artifacts/db-reset.js` (branch `fix/v4-auth`) | Semgrep | M1 PR |
| T5 | `isAdmin` middleware on `/benefits`; take `userId` from the session | `app/routes/index.js`, `app/routes/allocations.js` | Manual test / ZAP | Optional |
| T6 | Secrets moved to `.env` / GitHub Actions secrets (Vault bonus); Mongo credentials | `config/env/all.js`, `docker-compose.yml`, `.env.example` | Gitleaks | M1 |
| T7 | Remove the nested quantifier from the regex | `app/routes/profile.js` | Semgrep (`detect-non-literal-regexp` / manual) | Future work |
| T8 | Structured audit log of state-changing requests | Future work | none | Future work |
| T9 | Allow-list of research hosts | `app/routes/research.js` | Manual / ZAP | Future work |
| T10 | Only allow relative redirect paths | `app/routes/index.js` | Semgrep `express-open-redirect` | Future work |
| T11 | Upgrade base image; Trivy hard gate blocks HIGH/CRITICAL | `Dockerfile`, `.github/workflows/security-pipeline.yml` | Trivy, npm audit | M4 |
| All | Dependency and image scanning | `.github/workflows/security-pipeline.yml` | npm audit, Trivy | M4 |

## 5. Residual risk

- NodeGoat still uses `marked` 0.3.5 for memos; its `sanitize` option has known bypasses, so memo rendering stays a residual XSS risk until the dependency is upgraded (tracked under SCA / npm audit).
- The app runs over plain HTTP, so the session cookie is not marked `secure`. HTTPS termination is future work.
- T7 and T8 are documented but not fixed in this assignment.

## 6. Report text (paste into the report, ~480 words)

Table 1 = STRIDE table (section 2), Table 2 = risk matrix (section 3), Table 3 = threat-to-control mapping (section 4).


### Method

We used STRIDE, applied to each component and data flow in the architecture diagram **[FIG 1]**. We focused on the two trust boundaries: TB1 between the browser and the web container, and TB2 between the web container and MongoDB. For each threat we identified the exact code location, rated likelihood and impact on a 1-3 scale, and mapped it to a control and to the pipeline gate that would catch a regression.

### Key threats

We identified eleven application-specific threats (Table 1). Four map directly to the vulnerabilities the team exploited and fixed:

- **T1, Elevation of privilege:** the contributions form passes user input to `eval()`, allowing server-side code execution in the web container.
- **T2, Tampering:** the allocations `threshold` parameter is concatenated into a MongoDB `$where` expression, so input can alter the query and expose other users' data.
- **T3, Information disclosure / Spoofing:** profile names are stored and rendered without output encoding, so script saved by one user runs in the admin's browser on the Benefits page (stored XSS).
- **T4, Spoofing:** passwords are stored and compared in plaintext, so any database read becomes full account takeover.

The remaining threats cover missing role checks on admin routes (T5), hardcoded secrets and an unauthenticated database (T6), a regular expression vulnerable to catastrophic backtracking (T7, DoS), the absence of an audit trail (T8, Repudiation), server-side request forgery (T9), an open redirect (T10) and an end-of-life base image (T11).

### Risk assessment

Table 2 places each threat on a 3×3 likelihood × impact matrix. T1, T3 and T5 are rated **critical (9)**. Each needs only an ordinary account and a browser, and each leads to either server compromise or access to other employees' financial data. T2, T4, T6 and T11 are rated **high (6)**. The impact is severe, but each needs either specialised knowledge or a second foothold. T7 (DoS) and T9 (SSRF) are **medium (4)**: the first only affects availability, and the second can only reach the few services on the Docker network. T10 is **low (2)** because it needs a victim to click a crafted link. T8 is **medium (6)** because disputes over benefit changes are likely, but the harm is to accountability rather than confidentiality.

### Threat-to-control mapping

Table 3 links every threat to a concrete control and its location. For example, T3 is mitigated in `server.js` by enabling Swig auto-escaping, which is the primary output-encoding fix. It is backed by a Content-Security-Policy (`script-src 'self'`) set through helmet as defence in depth, so the browser refuses inline scripts even if encoding is missed somewhere **[FIG 8-10: before/after XSS and CSP header]**. Semgrep, npm audit, Gitleaks and Trivy in the pipeline act as regression controls for T1, T2, T6 and vulnerable dependencies respectively.

### Residual risk

Memo rendering still depends on `marked` 0.3.5, whose sanitiser has known bypasses. We record this as residual XSS risk to be closed by the dependency upgrade flagged by npm audit. The application also runs over HTTP, so the session cookie cannot yet be marked `secure`. Threats T7-T10 are documented as future work.
