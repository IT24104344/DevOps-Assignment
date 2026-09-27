# 1. Executive Summary & System Overview

*IE3142 DevOps Security – Building and Securing a DevSecOps Pipeline*
*Group repository: https://github.com/IT24104344/DevOps-Assignment*

## 1.1 Executive summary

This project builds and secures a working DevSecOps pipeline around **OWASP NodeGoat**,
an intentionally vulnerable open-source web application whose flaws are mapped to the
OWASP Top 10. We selected NodeGoat because it satisfies every requirement in the brief:
it is free and open-source, runs fully offline, ships an official `Dockerfile` and
`docker-compose.yml`, and is composed of **two separate communicating components** (a
Node.js/Express web service and a MongoDB database).

Our work covers: containerising and running the application, a STRIDE threat model with a
risk matrix and threat-to-control mapping, four exploit-and-fix demonstrations with
before/after SAST evidence, a GitHub Actions CI/CD pipeline with four automated security
gates (one configured to genuinely block the build), and secrets management that removes
hardcoded credentials from source in favour of GitHub Actions encrypted secrets.

## 1.2 Selected application

| Item | Detail |
|------|--------|
| Application | OWASP NodeGoat ("RetireEasy" employee retirement savings demo) |
| Source | https://github.com/OWASP/NodeGoat (unmodified baseline: commit `c5cb68a`) |
| Why chosen | Pre-vetted in the assignment's Appendix A; OWASP Top 10 mapped; strong STRIDE fit |
| Licence | Apache-2.0 (free/open-source) |

## 1.3 Technology stack

| Layer | Technology |
|-------|-----------|
| Runtime | Node.js 12 (Alpine base image) |
| Web framework | Express |
| Views | Swig templates |
| Database | MongoDB 4.4 |
| Orchestration | Docker + Docker Compose |
| CI/CD | GitHub Actions |
| Security tooling | Semgrep (SAST), npm audit (SCA), Gitleaks (secrets), Trivy (container) |

## 1.4 Containerisation approach

The application is brought up with a single command:

```bash
docker compose up -d --build
```

This starts two services defined in `docker-compose.yml`:

- **`web`** – built from the multi-stage `Dockerfile` (Node 12 Alpine). It waits for the
  database, seeds it via `artifacts/db-reset.js`, then runs `npm start`. Published on
  host port **4000**.
- **`mongo`** – the official `mongo:4.4` image, reachable only on the internal Compose
  network (port 27017 is *expose*d, not published to the host).

The two components communicate over the internal Docker network using the connection
string `mongodb://mongo:27017/nodegoat`, injected through the `MONGODB_URI` environment
variable rather than hardcoded.

## 1.5 Architecture diagram and trust boundaries

See `docs/architecture.svg`.

Three trust zones:

1. **Public internet** – the end user's browser, reaching the app over HTTP on port 4000.
   This is the untrusted zone; all input crossing into the app must be treated as hostile.
2. **Docker internal network** – the `web` container. The web/database boundary is crossed
   here; the database is never exposed to the host or internet.
3. **Data tier** – the `mongo` container, holding user accounts and allocation data.

The CI/CD pipeline sits alongside as a control plane: it builds the image and runs the four
security gates before code is considered shippable.

## 1.6 Repository layout (assignment artefacts)

| Path | Purpose |
|------|---------|
| `docs/01_SYSTEM_OVERVIEW.md` | This document (report §Executive Summary & System Overview) |
| `docs/02_THREAT_MODEL.md` | STRIDE threat model + risk matrix + threat-to-control (report §Threat Model) |
| `docs/architecture.svg` | Architecture diagram |
| `docs/SECRETS_MANAGEMENT.md` | Secrets management approach (report §Secrets Management) |
| `.github/workflows/security-pipeline.yml` | CI/CD pipeline with 4 security gates |
| `scans/baseline/` | Baseline (before-fix) Semgrep results |
| `NEED.md` | Evidence/screenshot capture guide for the final report |
| `CHECKLIST.md` | Master assignment checklist with status |
