# Architecture — x402-tech-risk

**Last synced:** 2026-08-29 · Scope of this doc: the `x402-tech-risk` repository only.

## What this repository is

This repo is a **distribution manifest**, not a service. It carries no worker code,
no CLI, and no tests — it exists to list the `tech-risk` MCP server on the MCP
registry and on Smithery, and to point clients at the remote endpoint.

Verified file inventory (all git-tracked files, `git ls-files`):

| File | Role |
|------|------|
| `README.md` | Human-facing usage + route table |
| `server.json` | MCP registry manifest (`io.github.rezearcher/tech-risk`, v1.2.0) |
| `smithery.yaml` | Smithery listing (remote runtime, streamable-http) |
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

## What the recent task changed

Task `t_5a57b5b0` (published the public GitHub repo + `smithery.yaml`) corresponds
to the single commit `a9008ba` "tech-risk MCP distribution manifest (smithery +
server.json + README)". That commit created the five files above. This is the
"last unexplored distribution surface" for the tech-risk play — the manifest layer.

## Gaps & unverified claims (do NOT treat as shipped)

These could not be proven from code in this repo and must be verified before any
downstream agent relies on them:

1. **Endpoint liveness / 402 gating — UNVERIFIED.** Could not probe
   `.../mcp`, `/enrich/tech-risk`, `/enrich/domain`, or `/scan/mcp` from this
   session (network calls need interactive approval). No proof here that the
   server responds, that discovery is free, or that paid routes return HTTP 402.
   Fleet state at sync time flagged `x402-data-api` as over its error budget
   (9/10 recent jobs failed) — a signal the worker may be degraded. Verify with a
   live `tools/list` POST and a 402 check before claiming the surface works.
2. **Registry listing — UNVERIFIED.** README states the server is "Listed on the
   official MCP registry ... (v1.0.0+)". Nothing in this repo proves the listing
   exists or is live; treat as a claim until confirmed against the registry.
3. **Repo publicity — UNVERIFIED.** The task title claims a *public* GitHub repo.
   Local git shows one commit; this doc did not confirm a remote or public
   visibility. Confirm on GitHub before asserting "published/public".
4. **Homepage link mismatch — CONFIRMED inconsistency.** `smithery.yaml`
   `homepage` and README's registry link point at
   `https://github.com/rezearcher/tech-risk`, but this repo's directory is
   `x402-tech-risk`. If the real public repo is `x402-tech-risk`, the published
   homepage link is broken. Reconcile the repo name vs. the advertised homepage.
5. **Version vagueness — CONFIRMED inconsistency.** `server.json` pins **1.2.0**
   while README says "**v1.0.0+**". Not a hard contradiction, but the two surfaces
   disagree on the stated version; align them.
6. **Implementation co-location — UNVERIFIED here.** The claim that tech-risk
   routes live inside `x402-data-api` (no separate worker) is asserted in the
   README but must be confirmed in that repo, not this one.
