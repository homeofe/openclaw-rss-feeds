# openclaw-rss-feeds — Log

## 2026-02-23 — P4 Finalization (Subagent)

Completed review, tests, docs, and repository push for plugin finalization.

### What was done
- Reviewed all `src/*.ts` modules with focus on runtime edge cases.
- Added defensive runtime handling in `src/index.ts` for missing or empty `feeds` config.
- Updated `src/notifier.ts` CLI argument from `--to` to `--target`.
- Added full test suite with Vitest:
  - `src/__tests__/fetcher.test.ts`
  - `src/__tests__/cveFetcher.test.ts`
  - `src/__tests__/formatter.test.ts`
  - `src/__tests__/ghostPublisher.test.ts`
- Rewrote `README.md` in English with:
  - install instructions
  - full config example with all options
  - cron and manual tool usage (`rss_run_digest`)
  - feed examples (Fortinet, Microsoft 365, BSI, Heise Security)
  - CVE enrichment section
  - Ghost integration section
- Updated AAHP files: `STATUS.md`, `LOG.md`, `NEXT_ACTIONS.md`.

### Validation
- `npx tsc --noEmit` passed (strict mode).
- `npm test` passed: **7/7 tests green**.


## 2026-02-22 — P4 Review (Opus — Architect)

**Critical fixes applied, APPROVED for v0.1.0-beta**

- Reviewed all 6 modules against ADR-001: all 6 decisions correctly implemented
- Plugin API usage (registerService + registerTool) verified against OpenClaw plugin spec
- **CRITICAL FIX:** Replaced `execSync` with `execFileSync` in notifier.ts — eliminates shell injection vector
- **CRITICAL FIX:** Added outer try/catch around Ghost publish block in index.ts — defensive non-fatal handling
- 5 minor issues documented (console.warn in cveFetcher, Fortinet-specific heuristics, missing plugin manifest, dryRun pattern, no RSS retry)
- `npx tsc --noEmit` → 0 errors after fixes
- Commit: 032b474

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

## 2026-02-22 — P4 Discussion Round (GPT Second Opinion)

- Independent review of the Sonnet implementation (Commit `42b2f04`) performed for:
  - `src/index.ts`
  - `src/fetcher.ts`
  - `src/notifier.ts`
  - `src/ghostPublisher.ts`
- Result documented in `.ai/handoff/REVIEW-GPT.md`.
- Positively assessed: module separation, error isolation, logging, Ghost JWT/API handling.
- Main criticism: `notifier.ts` uses `execSync` with a shell command string; despite escaping, this remains unnecessarily risky.
- Recommendation: switch to `spawn/execFile` with argument array + channel allowlist.
- Edge case check confirmed: empty feeds/no CVEs result in an empty but valid digest; Ghost failure is correctly handled as non-fatal and reflected in the notification.

## 2026-02-22 — Project Initialized

- Repo cloned from github.com/homeofe/openclaw-rss-feeds
- AAHP handoff structure created
- README.md written with architecture + config schema
- Context: Fortinet monthly digest was originally built in n8n, replaced by bash+Python cron script. Now generalizing as an OpenClaw plugin for any RSS feed + CVE enrichment. n8n has been shut down.
- Reference implementation: ~/.openclaw/workspace/cron/scripts/fortinet-monthly-digest.sh (540 lines, working)
