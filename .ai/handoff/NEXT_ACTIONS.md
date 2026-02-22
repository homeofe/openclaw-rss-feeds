# openclaw-rss-feeds — Next Actions

> Prioritized by strategic importance. Top = do first.

## P1 — Research (Sonar) ✅ DONE
- [x] Research: rss-parser npm package
- [x] Research: NVD CVE API v2.0 usage from TypeScript
- [x] Research: Ghost Admin API JWT from Node.js
- [x] Research: openclaw plugin scheduling API

## P2 — Architecture (Opus) ✅ DONE
- [x] ADR written: ADR-001 in `.ai/handoff/ADR.md`
- [x] Final config schema defined (feeds[], schedule, ghost, notify[], nvdApiKey?)
- [x] Dependencies finalized: rss-parser + jsonwebtoken + node-cron
- [x] Module structure defined (6 modules + types)
- [x] Error handling strategy documented

## P3 — Implementation (Sonnet) ✅ DONE — 2026-02-22
- [x] `src/types.ts` — Define interfaces: FeedConfig, CveEntry, FeedItem, DigestResult, PluginConfig
- [x] `src/fetcher.ts` — RSS fetch + parse with rss-parser, date filtering (lookbackDays), keyword filtering, dedup by link
- [x] `src/cveFetcher.ts` — NVD API v2.0 fetch, CVSS filtering (cvssThreshold), vendor CPE matching, 6s sleep between requests, optional apiKey header
- [x] `src/formatter.ts` — HTML digest builder: CVE severity table + feed items list, per-feed sections, summary header with date range
- [x] `src/ghostPublisher.ts` — JWT generation (HS256, id:secret split, 5min expiry, aud=/admin/), POST /ghost/api/admin/posts/?source=html
- [x] `src/notifier.ts` — Parse "channel:target" format, exec `openclaw message send --channel X --target Y --message Z`
- [x] `src/index.ts` — Plugin entry: registerService (node-cron if schedule set), registerTool "rss_run_digest" (manual trigger), wire all modules
- [x] `npm install` + `npx tsc --noEmit` — 0 errors (strict mode)
- [ ] Tests: fetcher (mock RSS XML), cveFetcher (mock NVD response), formatter (snapshot HTML output), ghostPublisher (mock JWT + fetch) — deferred to P4

## P4 — Review + Docs + Publish
- [ ] Discussion round (Opus reviews architecture compliance, ChatGPT second opinion)
- [ ] Update README.md with final config examples
- [ ] npm publish @elvatis/openclaw-rss-feeds
- [ ] Submit to OpenClaw community plugins
