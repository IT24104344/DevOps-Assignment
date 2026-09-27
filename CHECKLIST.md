# IE3142 DevSecOps Project: Group Checklist

Deadline: **Wed 1 Oct 2026** · App: **OWASP NodeGoat** (Node/Express + MongoDB) · Repo: https://github.com/IT24104344/DevOps-Assignment
M1 = Raj (lead, IT24104344) · M2 = Threat model + XSS · M3 = SAST / eval injection · M4 = CI/CD + NoSQL injection
Status: `[x]` done · `[~]` in progress / waiting on a person · `[ ]` not started. Screenshots to take are listed in **`NEED.md`** (E-numbers below).

_Last updated: 27 Sep 2026._

---

## Phase 0: Start
- [x] All: App chosen: OWASP NodeGoat (web + MongoDB, Apache-2.0, has Dockerfile + compose)
- [ ] All: Record the 4 names + student IDs + roles (needed on the title page)
- [x] M1: Ethical Clearance Form filled in
- [x] All: Ethics form signed by all 4 (include it in the submission, NEED E00)
- [x] M1: Unmodified NodeGoat imported into the group repo (`ae8b441`)
- [x] M1: M2, M3, M4 added as collaborators
- [ ] M1: Branch protection on `main` (require PR + passing checks). Optional but good for the viva.
- [ ] All: Each member clones the repo

## Phase 1: Run the app in Docker (§2.1, 3 marks)
- [x] Dockerfile present (multi-stage, runs as non-root `node`)
- [x] `docker-compose.yml` with `web` + `mongo` on a private network
- [x] `docker compose up -d --build` verified on Raj's laptop: both containers Up, login page loads
- [ ] M2, M3, M4: each confirms it runs on their own PC
- [ ] M1: Screenshots E01-E03

## Phase 2: Baseline scans (before any fix)
- [x] Baseline Semgrep saved: `scans/baseline/semgrep.json` + `semgrep.txt` (21 findings: 3 ERROR, 18 WARNING) (`fc2d428`)
- [x] Rule IDs noted: `code-string-concat` (V1 eval), `express-cookie-session-*`, `express-open-redirect`
- [~] M4: npm audit + Trivy "before" output. The pipeline uploads both as artifacts on its first run. Download them into `scans/baseline/`.
- [ ] M3: Screenshot E04

## Phase 3: Architecture + threat model (§2.2, 5 marks), M2
- [x] Architecture diagram with trust boundaries TB1/TB2 and numbered data flows: `docs/architecture.png` (+ `.svg`)
- [x] STRIDE table with 11 app-specific threats (T1-T4 match the team's 4 vulnerabilities): `docs/02_THREAT_MODEL.md`
- [x] 3×3 risk matrix with likelihood, impact and a justification per threat
- [x] Threat-to-control table with file paths and pipeline gates
- [ ] M2: fill the "Evidence" column with PR links once each fix is merged
- [x] Report text for this section (~480 words) at the end of `docs/02_THREAT_MODEL.md`

## Phase 4: Exploit → fix → re-test (§2.3, 6 marks), each on its own branch
Take every "(a) exploit" screenshot on unmodified `main` **before** merging any fix.
- [ ] M3: V1 eval injection, branch `fix/v1-eval`, PR (NEED E05)
- [ ] M4: V2 NoSQL injection, branch `fix/v2-nosql`, PR (NEED E06)
- [~] M2: V3 stored XSS, fix done on branch `m2/threat-model-xss` (autoescape + helmet CSP + httpOnly); PR open. Still need the screenshots E07a-f.
- [ ] M1: V4 broken auth (plaintext passwords → bcrypt), branch `fix/v4-auth`, PR (NEED E08)
- [ ] All: Screenshots saved in `evidence/` and committed from your own account
- [ ] M3: Review and merge all 4 PRs

## Phase 5: Secrets + Vault (§2.5)
- [x] Approach documented: `docs/SECRETS_MANAGEMENT.md`
- [ ] M1: Move `cookieSecret` / `cryptoKey` from `config/env/all.js` to env vars; add `.env.example`; `.env` in `.gitignore`
- [ ] M1: Add GitHub Actions repo secrets (E16)
- [ ] M1: (bonus) Vault dev container injecting the session secret at startup

## Phase 6: CI/CD pipeline (§2.4, 5 marks)
- [x] `.github/workflows/security-pipeline.yml` runs on every push/PR
- [x] Gate 1 SAST: Semgrep
- [x] Gate 2 SCA: npm audit
- [x] Gate 3 Secrets: Gitleaks
- [x] Gate 4 Container: Trivy, **hard gate** (`exit-code: 1` on HIGH/CRITICAL)
- [ ] M4: Screenshots E11-E13 from the Actions tab, including the red Trivy run (E12, required)
- [ ] M4: Pipeline diagram for the report
- [ ] M2: (optional, extra credit) OWASP ZAP baseline job

## Phase 7: After-scan evidence
- [ ] M3: Semgrep after all fixes merged (E10) + before/after table
- [ ] M4: npm audit + Trivy after

## Phase 8: Report (1800-2500 words)
- [~] System overview draft: `docs/01_SYSTEM_OVERVIEW.md`
- [x] Threat model section draft: `docs/02_THREAT_MODEL.md` §6
- [x] V3 secure-coding write-up: `docs/fixes/V3-stored-xss.md`
- [ ] M3: Secure-coding walkthrough for V1, V2, V4 + before/after SAST table
- [ ] M4: CI/CD design, 4 gates, blocked-build evidence
- [ ] M1: Secrets management section
- [ ] M4: Industry trends / case study (e.g. Log4Shell, Codecov). Easy marks, don't forget it.
- [ ] All: Reflection + future work
- [ ] All: Contribution statement + **AI-usage disclosure** (Claude was used for drafting code/docs), signed by all 4
- [ ] All: IEEE references
- [ ] M1: Word count check, export PDF

## Phase 9: Submit (Wed 1 Oct)
- [ ] README updated (setup, run, scan, secrets)
- [ ] Commit history shows all 4 members (E18)
- [ ] Module team can open the repo (it's public)
- [ ] M1: Submit on Courseweb: report PDF + signed ethics form + repo link
- [ ] All: Viva rehearsal (each explains their vulnerability, one threat and one gate in under 2 minutes)
