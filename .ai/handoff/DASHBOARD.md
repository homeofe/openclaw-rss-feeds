# openclaw-rss-feeds — Dashboard

> Phase: P3 ✅ (implementation done, ready for P4 review)

## Pipeline Status

| Phase | Agent  | Status      |
| ----- | ------ | ----------- |
| P1    | Sonar  | ✅ Done     |
| P2    | Opus   | ✅ Done     |
| P3    | Sonnet | ✅ Done     |
| P4    | All    | ⏳ Next     |

## Plugin Structure

| File                    | Status        | Notes                                     |
| ----------------------- | ------------- | ----------------------------------------- |
| README.md               | ✅ Done       |                                           |
| package.json            | ✅ Done       |                                           |
| tsconfig.json           | ✅ Done       |                                           |
| openclaw.plugin.json    | ✅ Done       |                                           |
| ADR.md                  | ✅ Done       |                                           |
| src/index.ts            | ✅ Implemented | registerService (cron) + registerTool     |
| src/types.ts            | ✅ Implemented | All shared interfaces                     |
| src/fetcher.ts          | ✅ Implemented | rss-parser, date filter, dedup, firmware  |
| src/cveFetcher.ts       | ✅ Implemented | NVD v2.0, CVSS filter, CPE match, 6s wait|
| src/formatter.ts        | ✅ Implemented | HTML tables, per-feed + combined CVE view |
| src/ghostPublisher.ts   | ✅ Implemented | JWT HS256, POST /ghost/api/admin/posts/   |
| src/notifier.ts         | ✅ Implemented | openclaw CLI subprocess, channel:target   |
| tests/*                 | ⏳ P4         | (deferred to review phase)                |

## TypeScript Status

```
npx tsc --noEmit → ✅ 0 errors (strict mode)
npm install      → ✅ 137 packages installed
```

## Key Design Decisions (from ADR-001)

- Pure TypeScript, no Python subprocess
- node-cron in `registerService` (optional) + manual `rss_run_digest` tool
- Sequential feed processing (NVD rate limits)
- All external failures are non-fatal (catch + log)
- Ghost JWT regenerated per request (5-min expiry)
- Notifications via `openclaw message send` CLI subprocess
