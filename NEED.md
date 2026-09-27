# NEED — Evidence & Screenshot Capture Guide

This file lists **every screenshot / piece of evidence** the group must capture for the
final Technical Report, and **exactly where and how** to capture each one. Work top to
bottom. Save each screenshot with the filename shown, under an `evidence/` folder.

> App URL when running locally: **http://localhost:4000**
> Start the app first with: `docker compose up -d --build`
> Default demo login is shown on the NodeGoat login/signup page (create a test user).

Legend: ☐ = to capture · ✅ = captured

---

## A. System overview & containerisation (Report §System Overview)

- ☐ **E01 — `evidence/E01_docker_compose_up.png`**
  Run `docker compose up -d --build`, then `docker compose ps`. Screenshot the terminal
  showing **both** `nodegoat-web` and `nodegoat-mongo` containers with status **Up**.
  *Proves: two communicating components, one-command bring-up.*

- ☐ **E02 — `evidence/E02_app_login_page.png`**
  Open http://localhost:4000 in a browser. Screenshot the "RetireEasy" login page.
  *Proves: the web component is serving.*

- ☐ **E03 — `evidence/E03_logged_in_dashboard.png`**
  Create a test account / log in. Screenshot the dashboard.
  *Proves: web ↔ MongoDB communication works (data is read/written).*

---

## B. Baseline SAST — BEFORE fixes (Report §Secure Coding, "before")

- ☐ **E04 — `evidence/E04_semgrep_baseline_summary.png`**
  Open `scans/baseline/semgrep.txt` and screenshot the summary block (total findings = 21,
  3 ERROR / 18 WARNING). *This is already generated and committed.*
  *Proves: the "before" SAST finding count.*

---

## C. Exploit → Fix → Re-test (Report §Secure Coding, LO2) — do 4 of these 5

For **each** vulnerability, capture THREE screenshots: (1) exploit works on the
unmodified app, (2) the code fix (diff), (3) the same attempt fails after the fix.
**Only run these after the Ethical Clearance Form is signed.**

### V1 — Server-side JS injection via `eval()`  (file: `app/routes/contributions.js:32-34`)
- ☐ **E05a — `evidence/E05a_v1_exploit.png`** — Log in → go to the **Contributions** page
  (http://localhost:4000/contributions). In the *preTax* field enter a JavaScript
  expression instead of a number and submit. Screenshot the result showing the expression
  was evaluated (e.g. an unexpected computed/large value).
- ☐ **E05b — `evidence/E05b_v1_fix_diff.png`** — Screenshot the code diff replacing `eval`
  with safe numeric parsing.
- ☐ **E05c — `evidence/E05c_v1_blocked.png`** — Repeat the exact same input; screenshot it
  now being rejected / treated as invalid.

### V2 — NoSQL injection via `$where`  (file: `app/data/allocations-dao.js:78`)
- ☐ **E06a — `evidence/E06a_v2_exploit.png`** — On the **Allocations search / threshold**
  feature, submit a crafted `threshold` value that returns records that should not be
  visible. Screenshot the extra rows returned.
- ☐ **E06b — `evidence/E06b_v2_fix_diff.png`** — Screenshot the diff removing the `$where`
  string and using typed query operators.
- ☐ **E06c — `evidence/E06c_v2_blocked.png`** — Repeat; screenshot no unauthorized rows.

### V3 — Stored XSS in profile  (file: `app/routes/profile.js:64` + `app/views/profile.html`)
- ☐ **E07a — `evidence/E07a_v3_exploit.png`** — Edit **Profile**
  (http://localhost:4000/profile), put a script payload in *First Name*, save, reload.
  Screenshot the script executing (e.g. an alert box).
- ☐ **E07b — `evidence/E07b_v3_fix_diff.png`** — Screenshot the diff adding server-side
  validation and output escaping.
- ☐ **E07c — `evidence/E07c_v3_blocked.png`** — Repeat; screenshot the payload shown as
  plain text, not executing.

### V4 — SSRF in stock research  (file: `app/routes/research.js:15-16`)
- ☐ **E08a — `evidence/E08a_v4_exploit.png`** — On the **Research** feature, set the `url`
  parameter to an internal address and screenshot the server returning internal content.
- ☐ **E08b — `evidence/E08b_v4_fix_diff.png`** — Screenshot the diff adding a host allow-list.
- ☐ **E08c — `evidence/E08c_v4_blocked.png`** — Repeat; screenshot the request being refused.

### V5 (spare) — Open redirect  (file: `app/routes/index.js:72`)
- ☐ **E09a/b/c** — `.../learn?url=` pointing off-site redirects → fix to allow-list → blocked.

### After-fix SAST
- ☐ **E10 — `evidence/E10_semgrep_after_fix.png`** — Re-run Semgrep after the fixes:
  `docker run --rm -v "%cd%:/src" -w /src semgrep/semgrep semgrep scan --config p/javascript --config p/nodejs --config p/owasp-top-ten --exclude scans --exclude node_modules .`
  Screenshot the new (lower) finding count. *Proves: before/after SAST diff.*

---

## D. CI/CD pipeline & security gates (Report §CI/CD, LO3)

- ☐ **E11 — `evidence/E11_actions_all_jobs.png`** — After pushing, open the repo's
  **Actions** tab → the "DevSecOps Security Pipeline" run. Screenshot the job list showing
  all four gates: Semgrep, npm audit, Gitleaks, Trivy.
- ☐ **E12 — `evidence/E12_gate_blocking_build.png`** — Screenshot the **Trivy (HARD GATE)**
  job **failing (red X)** on HIGH/CRITICAL vulnerabilities — this is the required
  "gate blocks the build" evidence.
- ☐ **E13 — `evidence/E13_semgrep_job_log.png`** — Open the Semgrep job log; screenshot the
  findings output.
- ☐ **E14 — `evidence/E14_pipeline_green.png`** *(optional, for the "runs green" checkpoint)*
  — Screenshot a run where the non-blocking gates pass on a clean commit.

---

## E. Secrets management (Report §Secrets Management)

- ☐ **E15 — `evidence/E15_gitleaks_finding.png`** — Screenshot the Gitleaks job detecting the
  hardcoded secret(s) (`cookieSecret` / `cryptoKey`) — motivates the migration.
- ☐ **E16 — `evidence/E16_github_secrets_settings.png`** — In repo **Settings → Secrets and
  variables → Actions**, screenshot the encrypted secrets you added (values are masked).
- ☐ **E17 — `evidence/E17_secret_injected_at_runtime.png`** — Screenshot the workflow using
  `${{ secrets.* }}` and/or the app reading the secret from an env var (not from source).

---

## F. Contribution evidence (Report §Individual Contribution)

- ☐ **E18 — `evidence/E18_commit_history_4_members.png`** — Screenshot the repo's commit
  history / contributors graph showing commits from **all 4 members**.
  *(Each member must push at least some commits under their own GitHub account.)*

---

## Quick capture order (recommended)
1. E01–E03 (app running) → 2. E04 (baseline SAST) → 3. push pipeline, capture E11–E15 →
4. add GitHub secrets, capture E16–E17 → 5. after ethics form: V1–V4 exploits E05–E09 →
6. E10 (after-fix SAST) → 7. E18 (contributions) at the end.
