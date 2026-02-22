# openclaw-rss-feeds — Log

## 2026-02-22 — P3 Implementation (Sonnet)

**6 modules implemented — TypeScript strict mode, 0 errors**

### Files Created
- `src/types.ts` — All shared interfaces: PluginConfig, FeedConfig, GhostConfig, FeedItem, FirmwareEntry, CveEntry, FeedResult, DigestResult, PluginApi, PluginService, PluginTool
- `src/fetcher.ts` — RSS/Atom feed fetcher using `rss-parser`. Features: date window filtering, keyword matching, dedup via Set (title::link key), firmware detection + version extraction, Fortinet docs URL generation, product sorting
- `src/cveFetcher.ts` — NVD CVE API v2.0 client. Features: per-keyword requests, CVSS score extraction (V3.1 → V3.0 → V2 fallback), CPE-based vendor matching, 6s rate-limit sleep, optional apiKey header, all errors caught (returns [])
- `src/formatter.ts` — HTML digest builder with two exports: `formatDigest` (full HTML doc) and `formatDigestBody` (Ghost-ready fragment). Features: per-feed sections (firmware table + CVE table + news items), summary header, combined CVE view for multi-feed digests, color-coded severity badges
- `src/ghostPublisher.ts` — Ghost Admin API draft publisher. Features: JWT HS256 with keyid (valid 5 min), hex secret decode from id:secret format, POST /ghost/api/admin/posts/?source=html, full error catch → returns {success, postId, postUrl, error}
- `src/notifier.ts` — Channel notifier via openclaw CLI subprocess. Features: channel:target format parsing, shell-safe escaping, execSync with 30s timeout, per-target error isolation, `buildDigestNotification` helper for consistent message format
- `src/index.ts` — Plugin entry. Features: `runDigest()` orchestrates all modules sequentially, `registerService` with node-cron (if schedule configured), `registerTool("rss_run_digest")` with optional dryRun flag, structured logging via api.logger

### Key Decisions Made During Implementation
- Used `formatDigestBody` (not full HTML doc) for Ghost to avoid duplicate `<html>` wrappers
- Added `enrichCve?: boolean` field to `FeedResult` type for formatter to conditionally show CVE section
- CommonJS module imports: no `.js` extensions (tsconfig `"module": "commonjs"` uses classic resolver)
- `AbortSignal.timeout()` for fetch timeouts (ES2020 target, available in Node 17.3+)
- Firmware dedup uses two keys: `title::link` for items AND `product-version` for firmware list

### TypeScript Check
```
npm install → 137 packages installed
npx tsc --noEmit → 0 errors (strict mode)
```

## 2026-02-22 — P2 Architecture Decision (Opus)

- **ADR-001** written: 6 architecture decisions documented
- D1: Pure TypeScript (no Python subprocess)
- D2: node-cron in registerService + manual tool (rss_run_digest)
- D3: Notification via openclaw CLI subprocess
- D4: CVE enrichment optional per-feed (enrichCve flag)
- D5: Sequential feed execution (NVD rate limit safety)
- D6: Final config schema: feeds[], schedule, lookbackDays, ghost{url,adminKey}, notify[], nvdApiKey?
- Config schema finalized in openclaw.plugin.json with uiHints for sensitive fields
- Dependencies: rss-parser + jsonwebtoken + node-cron (+ @types)
- Switched test runner from jest to vitest (faster, native ESM/TS)
- Module structure: 6 source files + types.ts
- Error strategy: per-feed isolation, non-fatal CVE/notification failures, structured logging

## 2026-02-22 — Project Initialized

- Repo cloned from github.com/homeofe/openclaw-rss-feeds
- AAHP handoff structure created
- README.md written with architecture + config schema
- Context: Fortinet monthly digest was originally built in n8n, replaced by bash+Python cron script. Now generalizing as an OpenClaw plugin for any RSS feed + CVE enrichment. n8n has been shut down.
- Reference implementation: ~/.openclaw/workspace/cron/scripts/fortinet-monthly-digest.sh (540 lines, working)
