# openclaw-rss-feeds — Log

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
