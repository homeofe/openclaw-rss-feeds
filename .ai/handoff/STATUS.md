# openclaw-rss-feeds — Status

> Last updated: 2026-02-22
> Phase: P1 — Ready for Sonar research + implementation

## Project Overview

**Package:** `@elvatis/openclaw-rss-feeds`
**Repo:** https://github.com/homeofe/openclaw-rss-feeds
**Purpose:** Generic RSS/Atom feed digest plugin — fetch, filter, enrich (CVE), draft, notify.

## Build Health

| Component           | Status     | Notes                                             |
| ------------------- | ---------- | ------------------------------------------------- |
| Repo / Structure    | (Verified) | Initialized + pushed 2026-02-22                   |
| Core logic (ref)    | (Verified) | cron/scripts/fortinet-monthly-digest.sh (working) |
| package.json        | (Verified) | Created, @elvatis/openclaw-rss-feeds              |
| tsconfig.json       | (Verified) | Created                                           |
| openclaw.plugin.json| (Verified) | Created, config schema defined                    |
| src/index.ts        | (Verified) | Stub created                                      |
| src/fetcher.ts      | (Unknown)  | Not yet implemented                               |
| src/cveFetcher.ts   | (Unknown)  | Not yet implemented                               |
| src/formatter.ts    | (Unknown)  | Not yet implemented                               |
| src/ghostPublisher.ts| (Unknown) | Not yet implemented                               |
| Tests               | (Unknown)  | Not yet created                                   |
| npm publish         | (Unknown)  | Not yet published                                 |

## Key Decision

Core logic already implemented in bash+Python (fortinet-monthly-digest.sh, ~540 lines, working in production).
Strategy: port to TypeScript modules, make all parameters configurable via openclaw.plugin.json.

## Reference Implementation

`~/.openclaw/workspace/cron/scripts/fortinet-monthly-digest.sh`

Modules to port:
1. NVD CVE fetch (keyword + date range + CVSS threshold)
2. RSS/Atom feed parse (rss-parser npm, date filtering, dedup)
3. HTML digest formatter
4. Ghost CMS draft creator (JWT HS256 auth)
5. Notification via openclaw message CLI or plugin API
