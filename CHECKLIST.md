# Master Assignment Checklist — IE3142 DevSecOps

Status: ✅ done · 🟡 in progress / needs your action · ☐ not started
Deadline: **1 October 2026**. Repo: https://github.com/IT24104344/DevOps-Assignment

> Note: I could not find the checklist your group already made (it isn't on this laptop and
> the project-files folder isn't reachable from here). Please send it and I'll merge the two.
> This is my reconstruction from the official brief + marking rubric.

## 2.1 Application, Architecture & Containerisation (3 marks)
- ✅ Select app meeting 2-component rule — OWASP NodeGoat (web + MongoDB)
- ✅ Each component containerised — official `Dockerfile` + `docker-compose.yml`
- ✅ One-command bring-up verified — `docker compose up -d --build` (both containers Up)
- ✅ Architecture diagram with trust boundaries — `docs/architecture.svg`
- 🟡 Capture E01–E03 screenshots (see `NEED.md`)

## 2.2 Threat Modelling & Risk Assessment (5 marks, LO2)
- ✅ STRIDE model, 6 app-specific threats — `docs/02_THREAT_MODEL.md`
- ✅ 5×5 likelihood/impact matrix with justifications
- ✅ Threat-to-control mapping with code/pipeline locations

## 2.3 Secure Coding: Exploit-and-Fix (6 marks, LO2)
- ✅ Baseline SAST (before) — `scans/baseline/` (21 findings)
- 🟡 Demonstrate 4 exploits working (needs signed ethics form) — V1–V4 in `NEED.md`
- ☐ Apply secure-coding fixes (I will write these once you confirm to proceed)
- ☐ Re-test each exploit — show blocked
- ☐ After-fix SAST count (E10)

## 2.4 CI/CD Pipeline & Security Automation (5 marks, LO3)
- ✅ GitHub Actions pipeline on every push — `.github/workflows/security-pipeline.yml`
- ✅ Gate 1 SAST (Semgrep)
- ✅ Gate 2 Dependency scan (npm audit)
- ✅ Gate 3 Secrets scan (Gitleaks)
- ✅ Gate 4 Container scan (Trivy) — configured as HARD GATE (fails build on HIGH/CRITICAL)
- 🟡 Capture a run where the gate blocks the build (E12) — after first push
- ☐ (optional, extra credit) DAST with OWASP ZAP

## 2.5 Secrets Management
- ✅ Approach documented — `docs/SECRETS_MANAGEMENT.md`
- ☐ Remove hardcoded `cookieSecret`/`cryptoKey` from `config/env/all.js`
- ☐ Add GitHub Actions encrypted secrets (E16) — needs repo admin (you)
- ☐ (bonus) HashiCorp Vault runtime injection

## 3. Submission components
- 🟡 Ethical Clearance Form — you said it's signed; include the file in submission
- ✅ Source repo with code, Docker, CI/CD, README
- 🟡 Commit history from all 4 members (E18) — each member pushes their part
- ☐ Technical Report PDF (1800–2500 words) — I can draft from the docs/ files
  - ☐ Industry trends / case study analysis (3 marks)
  - ☐ Reflection (1 mark)
  - ☐ Contribution statement + AI disclosure (1 mark)
  - ☐ IEEE references (1 mark)
- ☐ Viva prep (25 marks — individual, your team)

## Immediate actions for YOU
1. Send me the checklist your group already made so I can merge it.
2. Confirm: shall I (a) write the 4 secure-coding fixes now, and (b) migrate the secrets?
3. Add the 3 members as repo collaborators (for the 4-member commit history).
4. Add GitHub Actions secrets (COOKIE_SECRET, CRYPTO_KEY) once I push the secrets change.
