# ADR-001: openclaw-rss-feeds Architecture

## Context

We're porting the working Fortinet monthly digest (bash+Python, ~540 lines) into a generic OpenClaw plugin. The plugin must: fetch RSS/Atom feeds, optionally enrich with NVD CVE data, format an HTML digest, publish to Ghost CMS as draft, and notify via messaging channels.

Sonar research (P1) confirmed: `rss-parser`, `jsonwebtoken`, and `node-cron` are the right npm packages. OpenClaw plugin API supports `registerService` (background) and `registerTool` (agent-callable).

## Decisions

### D1: Runtime — Pure TypeScript vs Python subprocess
**Decision:** Pure TypeScript.
**Rationale:** All required functionality (RSS parsing, JWT signing, HTTP fetch, HTML templating) is available as npm packages. No Python dependency = simpler deployment, single runtime, no subprocess overhead. The bash script's Python usage was only for JWT + NVD fetch — both trivial in TS.

### D2: Scheduling — node-cron vs manual-only vs Gateway cron
**Decision:** Both: `node-cron` in `registerService` when `config.schedule` is set, plus a `rss_run_digest` agent tool for manual/on-demand execution.
**Rationale:** OpenClaw has no built-in `registerCron`. `node-cron` is lightweight and battle-tested. Making schedule optional means the plugin works both as automated background service and as an on-demand tool. If schedule is empty/missing, only the manual tool is available.

### D3: Notification — openclaw CLI subprocess vs plugin RPC
**Decision:** openclaw CLI subprocess (`openclaw message send`).
**Rationale:** The plugin API doesn't expose a direct message-sending RPC. CLI subprocess is the proven pattern (used by cron scripts today), requires no internal API coupling, and supports all configured channels (WhatsApp, Discord, Telegram, etc.) out of the box. Notification targets are configured per-plugin in `config.notify[]`.

### D4: NVD CVE enrichment — required vs optional
**Decision:** Optional, per-feed via `enrichCve: true` flag.
**Rationale:** Not every RSS feed needs CVE enrichment (e.g., a WordPress blog feed). NVD API has rate limits (5 req/30s without key). Making it opt-in per feed keeps the plugin generic. `nvdApiKey` in config is also optional — works without it at lower rate limits, which is fine for monthly runs.

### D5: Multi-feed execution — parallel vs sequential
**Decision:** Sequential with configurable delay.
**Rationale:** NVD rate limits (5 req/30s without key) make parallel CVE fetching risky. Sequential execution is simpler to debug, log, and error-handle. For feeds without CVE enrichment, the overhead is negligible (RSS fetch is fast). A future optimization could parallelize non-CVE feeds, but YAGNI for v0.1.

### D6: Ghost URL + config structure (final schema)
**Decision:** Ghost config as a dedicated object with `url` and `adminKey`. Full config schema below.

**Final config schema:**
```json
{
  "feeds": [
    {
      "id": "fortinet-firmware",
      "name": "Fortinet Firmware",
      "url": "https://support.fortinet.com/rss/firmware.xml",
      "keywords": ["FortiGate", "FortiAnalyzer"],
      "enrichCve": true,
      "cvssThreshold": 6.5,
      "tags": ["fortinet", "security"]
    }
  ],
  "schedule": "0 9 1 * *",
  "lookbackDays": 31,
  "ghost": {
    "url": "https://blog.example.com",
    "adminKey": "id:secret"
  },
  "notify": ["whatsapp:+49xxxxxxxxxx", "discord:#security"],
  "nvdApiKey": "optional-key"
}
```

**Rationale:** Flat `ghost` object keeps config simple. `adminKey` as `id:secret` matches Ghost's format directly. `notify` as string array with `channel:target` format is flexible and matches openclaw CLI syntax. `lookbackDays` replaces hardcoded "last month" logic — defaults to 31. `tags` per feed allows Ghost post tagging.

## Module structure

```
src/
  index.ts          — Plugin entry, registerService + registerTool
  fetcher.ts        — RSS fetch + parse (rss-parser), date filtering, dedup
  cveFetcher.ts     — NVD CVE API v2.0 fetch, CVSS filtering, vendor matching
  formatter.ts      — HTML digest builder (CVE table + feed items)
  ghostPublisher.ts — Ghost JWT generation (jsonwebtoken) + draft creation (fetch)
  notifier.ts       — Channel notification via openclaw CLI subprocess
  types.ts          — Shared TypeScript interfaces (FeedConfig, CveEntry, DigestResult, etc.)
```

## Dependencies

- `rss-parser` ^3.13.0 — RSS/Atom parsing
- `jsonwebtoken` ^9.0.0 + `@types/jsonwebtoken` — Ghost Admin API JWT
- `node-cron` ^3.0.0 + `@types/node-cron` — Background scheduling

## Error handling strategy

1. **Per-feed isolation:** Each feed runs in its own try/catch. A failing feed logs the error and continues to the next feed. The digest includes successfully fetched feeds only.
2. **NVD failure is non-fatal:** If CVE enrichment fails (network error, rate limit), the feed section is generated without CVE data, with a warning note in the digest.
3. **Ghost failure triggers notification:** If draft creation fails, the HTML digest is saved locally as fallback and a warning is sent via notification channels.
4. **Notification failure is logged only:** If `openclaw message send` fails, it's logged but doesn't block the pipeline.
5. **Structured logging:** All operations use `api.logger` (info/warn/error) for consistent OpenClaw log integration.
6. **Return value:** The `rss_run_digest` tool returns a structured result (feeds processed, CVEs found, Ghost URL or error) so the agent can report status.
