# Architecture — x402-tech-risk

**Last synced:** 2026-08-31 · Scope of this doc: the `x402-tech-risk` repository only.
**Distribution status:** all four distribution claims recorded PASS in
[`VERIFICATION.md`](VERIFICATION.md) (live probes 2026-08-30). Live-registry/endpoint
status is time-sensitive and **orchestrator-verified** — this doc reflects the recorded
probe results; the orchestrator re-curls the registries to confirm they are still live.

## What this repository is

This repo is a **distribution manifest**, not a service. It carries no worker code,
no CLI, and no tests — it exists to list the `tech-risk` MCP server on the MCP
registry and on Smithery, and to point clients at the remote endpoint.

Verified file inventory (all git-tracked files, `git ls-files`):

| File | Role |
|------|------|
| `README.md` | Human-facing usage + route table + Registry & Smithery sections |
| `server.json` | MCP registry manifest (`io.github.rezearcher/tech-risk`, v1.2.0) |
| `smithery.yaml` | Smithery listing (remote runtime, streamable-http) |
| `VERIFICATION.md` | Recorded live-probe verdicts for the four distribution claims (2026-08-30) |
| `ARCHITECTURE.md` | This doc |
| `LICENSE` | MIT |
| `.gitignore` | Secret/tooling excludes |

There is **no source code in this repo.** The implementation is claimed to live in
the separate [`x402-data-api`](https://github.com/rezearcher/x402-data-api) worker,
where the security routes (`/enrich/tech-risk`, `/enrich/domain`, `/scan/mcp`) are
co-located with the rest of that worker's x402 route portfolio — one wallet, one
facilitator, one deploy lifecycle. This co-location is a design assertion in the
README; it is **not verifiable from this repo** (the worker code is elsewhere).

## Manifest contents (verified by reading the files)

- **MCP remote endpoint:** `https://x402-data-api.sigrunner.workers.dev/mcp`
  (`streamable-http`) — declared identically in `server.json` and `smithery.yaml`.
- **Registry name:** `io.github.rezearcher/tech-risk`, version **1.2.0** in
  `server.json`.
- **Paid routes (per README):** `GET /enrich/tech-risk`, `GET /enrich/domain`,
  `GET /scan/mcp`, each $0.05 USDC on Base via x402; discovery is free.
- **Payment recipient:** `0x5765ae06a52dc7A0BB71c36A11db512c7ea9ed10` (README).

## What the recent tasks changed

The manifest layer was created by task `t_5a57b5b0` (commit `a9008ba`, "tech-risk MCP
distribution manifest"). Three follow-up tasks then verified and extended the
distribution surface — all landed as **docs/verification commits, not code** (this repo
still ships no worker/CLI/tests):

- **`t_7ae2053e` — verify distribution claims.** Ran live network probes and recorded
  the results in the new `VERIFICATION.md` (session git log: commit `778dc7f` "docs:
  verification of distribution claims (2026-08-30) — 3 PASS, 1 FAIL"). Endpoint
  liveness, 402 gating, MCP-registry listing, and repo publicity all PASS; the Smithery
  listing was the one FAIL at that point (never published).
- **`t_057daf23` — publish tech-risk on Smithery.** `smithery.yaml` already existed but
  had no live listing. The listing was published via the Smithery HTTP API (`PUT
  /servers/rezearcher/tech-risk` + `.../releases`, release
  `3f1c5287-fa71-4edf-99bd-f7eb72a37d8f` status SUCCESS, 22 tools discovered) and
  VERIFICATION.md's claim 4 flipped to PASS (session git log: commits `d05fc38`,
  `2b62405`).
- **`t_00c1fca6` — LIVE registry listing / canonical slug.** Re-verified the Smithery
  registry entry live and recorded the canonical namespaced slug `rezearcher/tech-risk`
  in both `VERIFICATION.md` and the new README **Smithery** section — a bare
  single-segment `tech-risk` slug 404s by Smithery's namespacing design and is not a
  missing listing (session git log: commit `93add43`).

**Commit-verification caveat:** the commit SHAs above are read from this session's
git log; this doc-sync task cannot run `git` to independently confirm them, so the
orchestrator cross-checks. They are cited as recorded history, not as fresh proof.

## Verified distribution facts (per VERIFICATION.md, 2026-08-30 probes)

Formerly-unverified claims that are now recorded PASS. Live-registry/endpoint status is
time-sensitive → **orchestrator-verified** (orchestrator re-curls); this doc reflects
the recorded probe evidence.

1. **Endpoint liveness / 402 gating — PASS (recorded).** `POST /mcp tools/list` → HTTP
   200 with 3 free-discovery tools; `GET /enrich/tech-risk`, `/enrich/domain`,
   `/scan/mcp` each → HTTP 402 + x402 `payment-required` challenge (paid routes not free).
   The ARCHITECTURE-sync-time fleet flag (`x402-data-api` over error budget) did **not**
   match live behavior — the worker responded and gated correctly.
2. **MCP registry listing — PASS (recorded).** `io.github.rezearcher/tech-risk` v1.2.0
   listed, status `active`, `isLatest: true` on the official MCP registry.
3. **Repo publicity — PASS (recorded).** `github.com/rezearcher/tech-risk` is public
   (`private: false`), not archived, default branch `main`.
4. **Smithery listing — PASS (recorded).** Live at
   `https://smithery.ai/server/rezearcher/tech-risk`, deployment proxy
   `https://tech-risk--rezearcher.run.tools` (x402-gated). Canonical slug
   `rezearcher/tech-risk`.

## Remaining gaps & doc drift (not shipped / not proven here)

1. **Implementation co-location — UNVERIFIED here.** The claim that tech-risk routes
   live inside the `x402-data-api` worker (no separate deployment) is asserted in the
   README but lives in a different repo; it cannot be proven from this repo.
2. **Challenge header-name drift — CONFIRMED.** README says unpaid calls return an
   `x-402-challenge` header, but the live probes (VERIFICATION.md) observed the header
   named `payment-required`. Same mechanism, wrong name in the README — align it.
3. **Version wording drift — MINOR.** `server.json` pins **1.2.0** while README says
   "**v1.0.0+**". Not a contradiction (1.0.0-and-up), but the two surfaces state the
   version differently; consider aligning.
4. **Resolved (was a gap):** the ARCHITECTURE-2026-08-29 "homepage link mismatch"
   concern is closed — the real public repo is `rezearcher/tech-risk` (the local
   checkout is just named `x402-tech-risk`), so the advertised homepage link is correct.
