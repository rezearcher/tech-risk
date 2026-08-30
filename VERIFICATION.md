# Verification — x402-tech-risk distribution claims

**Verified:** 2026-08-30 · Method: live network probes (curl, no auth, no spend)
**Scope:** claims marked UNVERIFIED in `ARCHITECTURE.md` (2026-08-29 sync), per task t_7ae2053e.
**Rule:** probing stopped at receipt of each 402 challenge; no x402 payment was settled.

---

## Verdict lines (canonical, one per claim)

PASS: Endpoint liveness + 402 gating — POST tools/list https://x402-data-api.sigrunner.workers.dev/mcp returned HTTP 200 with 3 free-discovery tools; GET /enrich/tech-risk, /enrich/domain, /scan/mcp each returned HTTP 402 with x402 payment-required challenge (not free) — 2026-08-30T07:16:49Z..07:17:03Z

PASS: MCP registry listing — https://registry.modelcontextprotocol.io/v0/servers?search=tech-risk returned HTTP 200 listing io.github.rezearcher/tech-risk v1.2.0 status=active isLatest=true — 2026-08-30T07:17:22Z

PASS: Repo publicity — https://api.github.com/repos/rezearcher/tech-risk returned HTTP 200, private=false, archived=false (public, reachable) — 2026-08-30T07:17:35Z

FAIL: smithery.yaml listing state — https://smithery.ai/server/tech-risk returned HTTP 200 soft-404 ("404: Page Not Found") and https://registry.smithery.ai/servers/@rezearcher/tech-risk returned HTTP 404 {"error":"Server not found"}; no live Smithery listing under any slug — 2026-08-30T07:17:44Z

---

## Detail per claim

### Claim 1 — Endpoint liveness + 402 gating: **PASS**

- `POST /mcp` (free discovery, `tools/list`) → **HTTP 200**, returned 3 tools
  (`enrich_tech_risk`, `enrich_domain`, `scan_mcp_server`).
  - URL: `https://x402-data-api.sigrunner.workers.dev/mcp` · status 200 · `2026-08-30T07:16:49Z`
- `GET /enrich/tech-risk?domain=example.com` → **HTTP 402** + `payment-required` x402 challenge header (`eyJ4ND...19fQ==`).
  - URL: `https://x402-data-api.sigrunner.workers.dev/enrich/tech-risk?domain=example.com` · status 402 · `2026-08-30T07:17:02Z`
- `GET /enrich/domain?domain=example.com` → **HTTP 402** + `payment-required` challenge header.
  - URL: `https://x402-data-api.sigrunner.workers.dev/enrich/domain?domain=example.com` · status 402 · `2026-08-30T07:17:02Z`
- `GET /scan/mcp?url=https://example.com/mcp` → **HTTP 402** + `payment-required` challenge header.
  - URL: `https://x402-data-api.sigrunner.workers.dev/scan/mcp?url=https://example.com/mcp` · status 402 · `2026-08-30T07:17:03Z`

Result: paid routes are **not reachable for free** — all three return the x402 payment
challenge and no data. Discovery is free. Fleet error-budget flag at ARCHITECTURE sync time
does not match live behavior: worker responds and gates correctly.

Note (doc drift, not a FAIL): README/ARCHITECTURE say `x-402-challenge` header; live header is
`payment-required`. Same challenge mechanism, different header name — align docs.

## Claim 2 — MCP registry listing: **PASS**

`io.github.rezearcher/tech-risk` **v1.2.0** is listed, status **active**, `isLatest: true`,
published `2026-07-17T01:00:14Z`. Older versions 0.1.0 (deprecated), 1.0.0, 1.1.0 (active) also present.

- URL: `https://registry.modelcontextprotocol.io/v0/servers?search=tech-risk` · status 200 · `2026-08-30T07:17:22Z`

## Claim 3 — Repo publicity: **PASS**

`github.com/rezearcher/tech-risk` is **public** (`private: false`), not archived, default
branch `main`, last push `2026-08-29T14:04:20Z`.

- URL: `https://api.github.com/repos/rezearcher/tech-risk` · status 200 · `2026-08-30T07:17:35Z`

Note: the public repo is named `tech-risk` (not `x402-tech-risk`). The smithery.yaml `homepage`
and README registry link point at `https://github.com/rezearcher/tech-risk` — which is the real,
public repo — so the ARCHITECTURE.md "homepage link mismatch" concern is resolved: the advertised
link is correct. Local directory name `x402-tech-risk` is just a local checkout name.

## Claim 4 — smithery.yaml listing state: **FAIL**

`smithery.yaml` exists in the repo (runtime: remote, streamable-http → the live MCP URL), but
**no live Smithery listing exists** for tech-risk under any slug:

- `https://smithery.ai/server/tech-risk` → HTTP 200 but page renders **"404: Page Not Found"** (soft-404 SPA) · `2026-08-30T07:17:44Z`
- `https://smithery.ai/server/x402-tech-risk` → HTTP 200 soft-404 · `2026-08-30T07:17:52Z`
- `https://smithery.ai/server/rezearcher-tech-risk` → HTTP 200 soft-404 · `2026-08-30T07:17:52Z`
- `https://registry.smithery.ai/servers/@rezearcher/tech-risk` → **HTTP 404** `{"error":"Server not found"}` · `2026-08-30T07:17:40Z`
- Smithery servers registry list: 0 hits for `tech-risk` / `rezearcher` in 10-server sample · `2026-08-30T07:17:47Z`

Result: the smithery.yaml manifest is present in the repo but the listing was never published /
never crawled. Any downstream claim of "listed on Smithery" is false. This is the sole FAIL.

---

## Summary

| Claim | Verdict | Evidence anchor |
|-------|---------|-----------------|
| 1. Endpoint liveness + 402 gating | PASS | /mcp 200 free discovery; 3 paid routes all 402 + challenge |
| 2. MCP registry listing (v1.2.0) | PASS | registry API lists active v1.2.0, isLatest |
| 3. Repo publicity | PASS | GitHub API: public, not archived |
| 4. smithery.yaml listing state | FAIL | Smithery soft-404 + registry 404 "Server not found" |

Follow-up filed: Smithery publish of `tech-risk` (t_<created>) — the repo carries smithery.yaml
but nothing is live on Smithery.
