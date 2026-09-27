# 5. Secrets Management

*Report section: Secrets Management*

## 5.1 Problem in the baseline

The unmodified NodeGoat commits secrets directly into source:

- `config/env/all.js` → `cookieSecret: "session_cookie_secret_key_here"`
- `config/env/all.js` → `cryptoKey: "a_secure_key_for_crypto_here"`
- The MongoDB connection string is already read from `process.env.MONGODB_URI` (good), with a
  localhost fallback only for developer convenience.

Committed secrets are flagged automatically by the **Gitleaks** gate in the pipeline.

## 5.2 Approach

1. **Remove hardcoded values from source.** `cookieSecret` and `cryptoKey` are read from
   environment variables (`process.env.COOKIE_SECRET`, `process.env.CRYPTO_KEY`), with no
   real secret checked in.
2. **Local development** uses a git-ignored `.env` file (never committed). A committed
   `.env.example` documents the required variable names with placeholder values only.
3. **Pipeline / CI** uses **GitHub Actions encrypted secrets**
   (repo → Settings → Secrets and variables → Actions). The workflow references them as
   `${{ secrets.COOKIE_SECRET }}` etc., so plaintext never appears in source or logs.
4. **Runtime injection to the container** is done through Docker Compose environment
   variables sourced from the environment / `.env`, e.g. `MONGODB_URI`, `COOKIE_SECRET`,
   `CRYPTO_KEY` — mirroring how the existing `MONGODB_URI` is already injected.

## 5.3 Bonus (extra marks): HashiCorp Vault
For higher marks the brief rewards demonstrating a secrets manager injecting a secret at
runtime. Optional next step: run Vault in dev mode as a compose service, store `COOKIE_SECRET`
there, and have an entrypoint fetch it into the environment before `npm start`.

## 5.4 How secrets are provisioned (summary for the report)

| Consumer | Source of secret | Mechanism |
|----------|------------------|-----------|
| Local app | `.env` (git-ignored) | Docker Compose env vars |
| Running container | env vars | `environment:` block in `docker-compose.yml` |
| CI/CD pipeline | GitHub Actions encrypted secrets | `${{ secrets.* }}` in workflow |
| (bonus) Runtime | HashiCorp Vault | entrypoint fetch before start |
