# Tech Risk Enrichment (x402)

Pay-per-call security enrichment MCP server for AI agents: **tech-stack → CVE/EPSS/CISA-KEV findings and risk score**, plus **MCP-server scanning** for tool-poisoning, prompt-injection, and exfiltration risk (OWASP LLM01/LLM08).

- **MCP endpoint:** `https://x402-data-api.sigrunner.workers.dev/mcp` (streamable-http)
- **Price:** $0.05 USDC per query (Base mainnet), paid via [x402](https://x402.org) — no API key required
- **Payment recipient:** `0x5765ae06a52dc7A0BB71c36A11db512c7ea9ed10`
- **License:** MIT

## About this repository

This repository is the **distribution manifest** for the `io.github.rezearcher/tech-risk` MCP server (see [server.json](server.json) and [smithery.yaml](smithery.yaml)). The server itself is a remote worker: discovery (`tools/list`) and `tools/call` are free; paid HTTP routes (`/enrich/tech-risk`, `/enrich/domain`, `/scan/mcp`) return HTTP 402 with an x402 payment challenge, and the agent pays to unlock the response.

**Implementation source:** the tech-risk functionality is part of the [x402-data-api](https://github.com/rezearcher/x402-data-api) worker's route portfolio — there is deliberately **no separate worker deployment** for tech-risk. Security endpoints (tech-risk, domain, scan/mcp) are co-located in the x402-data-api worker so the whole surface shares one wallet, one facilitator, and one deployment lifecycle.

## MCP usage

Register the remote server in your MCP client:

```yaml
mcp_servers:
  x402-tech-risk:
    url: "https://x402-data-api.sigrunner.workers.dev/mcp"
```

### Example: tech-risk enrichment

```
tools/call:
  name: enrich_tech_risk
  arguments:
    domain: example.com
```

Returns: detected tech stack → matched CVEs with EPSS scores, CISA-KEV status, and an aggregate risk verdict. Unpaid calls get `402` + `x-402-challenge` header; the agent completes the x402 flow to release the data.

### Example: MCP server scan

```
tools/call:
  name: scan_mcp
  arguments:
    url: https://some-mcp-endpoint/mcp
```

Scans a remote MCP server for tool-poisoning / prompt-injection / exfiltration risk and returns an OWASP-aligned risk report.

## Direct HTTP (paid)

| Route | Price | Description |
|-------|-------|-------------|
| `GET /enrich/tech-risk?domain=<d>` | $0.05 | Tech-stack → CVE/EPSS/CISA-KEV findings + risk score |
| `GET /enrich/domain?domain=<d>` | $0.05 | Domain enrichment |
| `GET /scan/mcp?url=<mcp-url>` | $0.05 | MCP server security scan (LLM01/LLM08) |

## Registry

Listed on the official MCP registry as `io.github.rezearcher/tech-risk` (v1.0.0+), remote → `https://x402-data-api.sigrunner.workers.dev/mcp`.
