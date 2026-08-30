# Verification — x402-tech-risk distribution claims

**Verified:** 2026-08-30 · Method: live network probes (curl, no auth, no spend)
**Scope:** claims marked UNVERIFIED in `ARCHITECTURE.md` (2026-08-29 sync), per task t_7ae2053e.
**Rule:** probing stopped at receipt of each 402 challenge; no x402 payment was settled.

---

## Verdict lines (canonical, one per claim)

PASS: Endpoint liveness + 402 gating — POST tools/list https://x402-data-api.sigrunner.workers.dev/mcp returned HTTP 200 with 3 free-discovery tools; GET /enrich/tech-risk, /enrich/domain, /scan/mcp each returned HTTP 402 with x402 payment-required challenge (not free) — 2026-08-30T07:16:49Z..07:17:03Z

PASS: MCP registry listing — https://registry.modelcontextprotocol.io/v0/servers?search=tech-risk returned HTTP 200 listing io.github.rezearcher/tech-risk v1.2.0 status=active isLatest=true — 2026-08-30T07:17:22Z

PASS: Repo publicity — https://api.github.com/repos/rezearcher/tech-risk returned HTTP 200, private=false, archived=false (public, reachable) — 2026-08-30T07:17:35Z

PASS: Smithery listing — https://smithery.ai/server/rezearcher/tech-risk returned HTTP 200 rendering "Tech Risk Enrichment" with all 22 tools; registry API https://api.smithery.ai/servers/rezearcher/tech-risk returned HTTP 200 (remote, deploymentUrl https://tech-risk--rezearcher.run.tools); release 3f1c5287-fa71-4edf-99bd-f7eb72a37d8f status=SUCCESS, scan discovered 22 tools — 2026-08-30T07:26Z (published via PUT /servers/rezearcher/tech-risk + PUT .../releases with external_shttp payload)

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

## Claim 4 — smithery.yaml listing state: **PASS** (now published)

`smithery.yaml` exists in the repo (runtime: remote, streamable-http → the live MCP URL). The
listing was **not live** at 07:17Z (soft-404s; never published), but was **published and verified
live** at 07:26Z via the Smithery HTTP API:

- `PUT https://api.smithery.ai/servers/rezearcher/tech-risk` → **HTTP 201** (server created) · `2026-08-30T07:23:58Z`
- `PUT https://api.smithery.ai/servers/rezearcher/tech-risk/releases` → **HTTP 202**, release
  `3f1c5287-fa71-4edf-99bd-f7eb72a37d8f` status **SUCCESS** (`external_shttp`, upstream
  `https://x402-data-api.sigrunner.workers.dev/mcp`); scan discovered server
  `x402-data-api v0.1.0` with **22 tools**; deploy log: "Deployment successful" · `2026-08-30T07:26:28Z..07:26:38Z`
- `https://api.smithery.ai/servers/rezearcher/tech-risk` → **HTTP 200**, `remote: true`,
  `deploymentUrl: https://tech-risk--rezearcher.run.tools`, `connections[0].type: http` with full
  22-tool list · `2026-08-30T07:27Z`
- `https://smithery.ai/server/rezearcher/tech-risk` → **HTTP 200** rendering the listing
  (displayName "Tech Risk Enrichment", tool names `enrich_tech_risk`, `scan_mcp_server`, etc.) ·
  `2026-08-30T07:27Z`
- Proxy root `https://tech-risk--rezearcher.run.tools` → **HTTP 401 x402 challenge**
  (`invalid_token` / `Missing Authorization header`) — i.e. the Smithery proxy is live and
  correctly enforces the x402 payment gate (NOT a 404; this is the designed pay-per-query behavior).

Result: `tech-risk` is now live on Smithery at `https://smithery.ai/server/rezearcher/tech-risk`,
with deployment `https://tech-risk--rezearcher.run.tools` proxying to the x402-gated upstream.

---

## Summary

| Claim | Verdict | Evidence anchor |
|-------|---------|-----------------|
| 1. Endpoint liveness + 402 gating | PASS | /mcp 200 free discovery; 3 paid routes all 402 + challenge |
| 2. MCP registry listing (v1.2.0) | PASS | registry API lists active v1.2.0, isLatest |
| 3. Repo publicity | PASS | GitHub API: public, not archived |
| 4. smithery.yaml listing state | PASS | live listing smithery.ai/server/rezearcher/tech-risk + SUCCESS release |

Follow-up (t_057daf23) resolved 2026-08-30T07:26Z: Smithery publish of `tech-risk` completed —
listing live at https://smithery.ai/server/rezearcher/tech-risk, proxy at
https://tech-risk--rezearcher.run.tools (x402-gated).

### Canonical slug note (t_00c1fca6 re-verify, 2026-08-30)

Re-verified live (registry HTTP 200 with full JSON, tools listed, deployment proxy up):
`https://registry.smithery.ai/servers/rezearcher/tech-risk` → **200**.

The canonical slug is `rezearcher/tech-risk` (namespaced). A bare single-segment
`https://registry.smithery.ai/servers/tech-risk` → 404 is NOT a missing listing:
Smithery namespaces third-party servers as `<owner>/<name>` (single-segment slugs like
`brave`/`exa` exist only for first-party bySmithery servers — confirmed in
https://smithery.ai/docs and by the 404 payload `{"error":"Namespace not found"}`).
Registering a duplicate under a second slug is explicitly against the task guardrail
(exactly one canonical entry), so the canonical slug is recorded here and in README.md.
