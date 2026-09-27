# NEED: Evidence and Screenshot Guide for the Final Report

Every screenshot the report needs, **who** takes it, **where** to take it, the **filename** to save it as, and **where it goes in the report**.
Save all screenshots in `evidence/` at the repo root and commit them **from your own GitHub account** (this also gives every member commits).

- App URL: **http://localhost:4000** · Start: `docker compose up -d --build` · Reset data: `docker compose down -v` then start again
- Demo logins (seeded by `artifacts/db-reset.js`): `admin / Admin_123`, `user1 / User1_123`, `user2 / User2_123`
- Exploit demos only on your own local Docker copy. The ethics form is signed; include it in the submission.
- ☐ = to capture · ✅ = captured. Tick them here as you go.

---

## Quick table

| ID | Owner | What | Report placement |
|---|---|---|---|
| E00 | M1 | Signed ethics form (scan/PDF, not a screenshot) | Submission + Appendix A |
| E01 | M1 | `docker compose ps`, both containers Up | §2 System Overview, Fig 2 |
| E02 | M1 | Login page in browser | §2 System Overview, Fig 3 |
| E03 | M1 | Logged-in dashboard | §2 System Overview, Fig 3 (side by side with E02) |
| E04 | M3 | Baseline Semgrep summary (21 findings) | §4 Secure Coding, Table 4 "Before" |
| E05a/b/c | M3 | V1 `eval()` injection: exploit / fix diff / blocked | §4.1 V1 |
| E06a/b/c | M4 | V2 NoSQL injection: exploit / fix diff / blocked | §4.2 V2 |
| E07a-f | M2 | V3 stored XSS: exploit / admin hit / source / fix diff / blocked / CSP header | §4.3 V3 |
| E08a/b/c | M1 | V4 broken auth: plaintext passwords / bcrypt diff / hashed passwords | §4.4 V4 |
| E09 | spare | SSRF or open redirect (only if a V1-V4 demo fails) | §4 (replacement) |
| E10 | M3 | Semgrep after all fixes | §4 Secure Coding, Table 4 "After" |
| E11 | M4 | Actions run showing all 4 gates | §5 CI/CD, Fig 5 |
| E12 | M4 | Trivy gate **failing** the build (red X) | §5 CI/CD, Fig 6 (**required**) |
| E13 | M4 | Semgrep job log in Actions | §5 CI/CD, Appendix B |
| E14 | M4 | A green run (optional) | §5 CI/CD, Fig 7 |
| E15 | M1 | Gitleaks detecting the hardcoded secret | §6 Secrets Management, Fig 8 |
| E16 | M1 | GitHub Actions secrets page (values masked) | §6 Secrets Management, Fig 9 |
| E17 | M1 | Secret read from env var / `${{ secrets.* }}` | §6 Secrets Management, Fig 10 |
| E18 | All | Commit history / contributors showing all 4 | §9 Contribution Statement |
| E19 | M2 | Architecture diagram (use `docs/architecture.png` directly) | §2 System Overview, Fig 1 |
| E20 | M2 | Threat model tables (copy from `docs/02_THREAT_MODEL.md`) | §3 Threat Model, Tables 1-3 |

---

## A. System overview and containerisation (§2)

- ☐ **E01 `evidence/E01_docker_compose_up.png`**: in a terminal in the repo folder run `docker compose up -d --build` then `docker compose ps`. Capture the terminal with **both** web and mongo containers showing **Up**. *Proves two communicating components with one-command bring-up.*
- ☐ **E02 `evidence/E02_app_login_page.png`**: open http://localhost:4000 and capture the "RetireEasy" login page with the address bar visible.
- ☐ **E03 `evidence/E03_logged_in_dashboard.png`**: log in as `user1`, capture the Dashboard. *Proves web ↔ MongoDB works.*
- ✅ **E19**: `docs/architecture.png` is ready. Insert it as Figure 1.

## B. Baseline SAST, before fixes (§4)

- ☐ **E04 `evidence/E04_semgrep_baseline_summary.png`**: open `scans/baseline/semgrep.txt` and capture the summary block (21 findings: 3 ERROR, 18 WARNING).

## C. Exploit → fix → re-test (§4, the biggest marks)

For each vulnerability capture: (a) it working on the **unmodified** app, (b) the code fix diff (GitHub PR "Files changed" view is ideal), (c) the same attempt failing after the fix.
Take (a) on `main` **before** merging the fix PR, then switch to the fix branch (`git checkout <branch>` then `docker compose up -d --build`) for (c).

### V1: Server-side JS injection via `eval()` (M3), `app/routes/contributions.js:32-34`
- ☐ **E05a `evidence/E05a_v1_exploit.png`**: Contributions page (http://localhost:4000/contributions). Follow NodeGoat's tutorial at http://localhost:4000/tutorial/a1 with a harmless expression and capture the evaluated result.
- ☐ **E05b `evidence/E05b_v1_fix_diff.png`**: diff replacing `eval` with `parseInt` and range checks.
- ☐ **E05c `evidence/E05c_v1_blocked.png`**: same input now rejected as invalid.

### V2: NoSQL injection via `$where` (M4), `app/data/allocations-dao.js`
- ☐ **E06a `evidence/E06a_v2_exploit.png`**: Allocations page, following http://localhost:4000/tutorial/a1 (NoSQL section). Capture rows belonging to other users appearing.
- ☐ **E06b `evidence/E06b_v2_fix_diff.png`**: diff removing the `$where` string and validating `threshold` as an integer.
- ☐ **E06c `evidence/E06c_v2_blocked.png`**: same request shows only your own rows or an "invalid threshold" error.

### V3: Stored XSS (M2), `server.js` (Swig `autoescape: false`), rendered in `layout.html` and `benefits.html`
Fix is on branch **`m2/threat-model-xss`**. Full steps are in `docs/fixes/V3-stored-xss.md`.
- ☐ **E07a `evidence/E07a_v3_exploit_self.png`**: on `main`, log in as `user1` → **Profile**. Put a harmless alert test script (from http://localhost:4000/tutorial/a3) in **First Name**. Set **Bank Routing #** to digits ending in `#` (e.g. `123456#`) or the save is rejected. Submit and reload. Capture the alert box.
- ☐ **E07b `evidence/E07b_v3_exploit_admin.png`**: log out, log in as `admin` → **Benefits**. Capture the alert firing in the **admin's** browser. This proves it is *stored* and hits another user.
- ☐ **E07c `evidence/E07c_v3_source_before.png`**: on that page, right-click → View page source → Ctrl+F `script`. Capture the raw, unescaped tag in the name cell.
- ☐ **E07d `evidence/E07d_v3_fix_diff.png`**: the PR "Files changed" view for `server.js` (autoescape + helmet CSP).
- ☐ **E07e `evidence/E07e_v3_blocked.png`**: `git checkout m2/threat-model-xss`, then `docker compose down -v` and `docker compose up -d --build`. Repeat E07a and E07b: **no alert**, the name shows as plain text. Capture the admin Benefits page.
- ☐ **E07f `evidence/E07f_v3_csp_header.png`**: F12 → Network → reload → click the first request → Response Headers. Capture the `Content-Security-Policy: default-src 'self'; script-src 'self' ...` line.

### V4: Broken authentication, plaintext passwords (M1), `app/data/user-dao.js`
- ☐ **E08a `evidence/E08a_v4_plaintext.png`**: on `main` run
  `docker compose exec mongo mongo nodegoat --quiet --eval "db.users.find({}, {userName:1, password:1}).forEach(printjson)"`
  and capture passwords stored in **plain text**.
- ☐ **E08b `evidence/E08b_v4_fix_diff.png`**: diff switching `addUser` / `validateLogin` to bcrypt (and hashed seed users in `artifacts/db-reset.js`).
- ☐ **E08c `evidence/E08c_v4_hashed.png`**: on the fix branch, reset the DB, run the same command, and capture `$2a$10$...` hashes. Also show login still works.

### Spare (only if one of V1-V4 fails)
- ☐ **E09 `evidence/E09_spare_*.png`**: SSRF in `app/routes/research.js` or open redirect in `app/routes/index.js:72`, with the same a/b/c pattern.

### After-fix SAST
- ☐ **E10 `evidence/E10_semgrep_after_fix.png`**: after all fix PRs are merged, run on `main` (Windows CMD in the repo folder):
  `docker run --rm -v "%cd%:/src" -w /src semgrep/semgrep semgrep scan --config p/javascript --config p/nodejs --config p/owasp-top-ten --exclude scans --exclude node_modules .`
  Capture the lower finding count and put before/after numbers in Table 4.

## D. CI/CD pipeline (§5)

- ☐ **E11 `evidence/E11_actions_all_jobs.png`**: GitHub → **Actions** → "DevSecOps Security Pipeline" run. Capture the job graph showing all 4 gates.
- ☐ **E12 `evidence/E12_gate_blocking_build.png`**: capture the **Gate 4 Trivy** job **failing** with HIGH/CRITICAL findings. This is the required "gate blocks the build" evidence.
- ☐ **E13 `evidence/E13_semgrep_job_log.png`**: open the Semgrep job log and capture the findings output.
- ☐ **E14 `evidence/E14_pipeline_green.png`** (optional): a run where the other gates pass.

## E. Secrets management (§6)

- ☐ **E15 `evidence/E15_gitleaks_finding.png`**: Gitleaks job flagging `cookieSecret` / `cryptoKey` in `config/env/all.js`.
- ☐ **E16 `evidence/E16_github_secrets_settings.png`**: repo **Settings → Secrets and variables → Actions**, showing the masked secrets.
- ☐ **E17 `evidence/E17_secret_injected_at_runtime.png`**: code or workflow reading the secret from `process.env` / `${{ secrets.* }}` instead of source.

## F. Contribution (§9)

- ☐ **E18 `evidence/E18_commit_history_4_members.png`**: repo **Insights → Contributors** (or the commits list) showing all 4 members.

---

## Suggested order
1. E01-E04 now (app is running) → 2. E11-E13, E15 from the first pipeline run → 3. exploit screenshots (a) for V1-V4 **on `main` before merging any fix** → 4. merge each fix PR and take (b) and (c) → 5. E10 → 6. secrets E16-E17 → 7. E18 last.
