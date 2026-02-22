# openclaw-rss-feeds — Research Notes (P1 Sonar)
Date: 2026-02-22

---

## 1. rss-parser npm

**Package:** `rss-parser` (npm, ~2M weekly downloads)
**Source:** https://www.npmjs.com/package/rss-parser

### API
```typescript
import Parser from 'rss-parser';

type CustomItem = { 'content:encoded': string };
const parser = new Parser<{}, CustomItem>({
  customFields: { item: ['content:encoded'] }
});

const feed = await parser.parseURL('https://support.fortinet.com/rss/firmware.xml');
// feed.title, feed.items[]
// item: { title, link, pubDate, content, isoDate, ... }
```

### Key facts
- Supports RSS 2.0 + Atom natively
- `parseURL(url)` and `parseString(xmlString)` methods
- TypeScript generics for custom fields (e.g. `media:content`, `content:encoded`)
- `item.isoDate` = ISO 8601 string (easy date filtering)
- `customFields: { feed: [], item: [] }` for non-standard namespaces
- `keepArray: true` for multi-value fields

### Recommendation
Use `rss-parser`. No alternative needed - it's the de-facto standard and TypeScript-native.

---

## 2. NVD CVE API v2.0

**Endpoint:** `https://services.nvd.nist.gov/rest/json/cves/2.0`
**Docs:** https://nvd.nist.gov/developers/vulnerabilities

### Key parameters
| Param           | Description                          |
| --------------- | ------------------------------------ |
| keywordSearch   | Text search (e.g. "fortinet")        |
| pubStartDate    | ISO 8601 (e.g. 2026-01-01T00:00:00) |
| pubEndDate      | ISO 8601                             |
| apiKey (header) | Optional, strongly recommended       |

### Rate limiting
- **Without API key:** 5 requests / 30 seconds
- **With API key:** 50 requests / 30 seconds
- **NIST recommendation:** sleep 6s between requests
- **Max results:** 2000 per request
- API key is FREE from nvd.nist.gov — just register

### TypeScript fetch
```typescript
const res = await fetch(
  `https://services.nvd.nist.gov/rest/json/cves/2.0?keywordSearch=fortinet&pubStartDate=${start}&pubEndDate=${end}`,
  { headers: apiKey ? { apiKey } : {} }
);
const data = await res.json();
const vulns = data.vulnerabilities ?? [];
// vuln.cve.id, vuln.cve.descriptions[].value
// vuln.cve.metrics.cvssMetricV31[0].cvssData.baseScore
// vuln.cve.configurations[].nodes[].cpeMatch[].criteria (CPE string, e.g. "cpe:2.3:a:fortinet:...")
```

### Recommendation
- Add optional `nvdApiKey` to plugin config
- Sleep 6s after each request (single monthly call = no issue)
- CPE matching for vendor-specific filtering: check `cpe:2.3:*:fortinet:` in criteria

---

## 3. Ghost Admin API JWT (Node.js)

**Package:** `jsonwebtoken` npm (`npm install jsonwebtoken @types/jsonwebtoken`)
**Ghost docs:** https://ghost.org/docs/admin-api/

### JWT generation
```typescript
import jwt from 'jsonwebtoken';

function makeGhostToken(adminKey: string): string {
  const [id, secret] = adminKey.split(':');
  const secretBytes = Buffer.from(secret, 'hex');
  const now = Math.floor(Date.now() / 1000);
  return jwt.sign(
    { iat: now, exp: now + 300, aud: '/admin/' },
    secretBytes,
    { algorithm: 'HS256', keyid: id }
  );
}
```

### Create post (draft)
```typescript
const token = makeGhostToken(process.env.GHOST_ADMIN_KEY!);
const res = await fetch(`${ghostUrl}/ghost/api/admin/posts/?source=html`, {
  method: 'POST',
  headers: {
    Authorization: `Ghost ${token}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    posts: [{ title, html, status: 'draft', tags: [...] }]
  }),
});
```

### Key notes
- Token expires in 5min — regenerate per request
- `aud` must be `/admin/` (not `/v4/admin/` — our bash script uses `/admin/` and works)
- `source=html` in query string = use HTML body instead of mobiledoc

### Recommendation
Use `jsonwebtoken`. No Ghost JS SDK needed — plain fetch works fine.

---

## 4. OpenClaw Plugin Scheduling

**Finding:** OpenClaw plugins do NOT have a `api.registerCron()` method.

### Available scheduling options from a plugin

**Option A — `api.registerService()` + node-cron**
```typescript
import cron from 'node-cron';

api.registerService({
  id: 'rss-digest-scheduler',
  start() {
    cron.schedule(config.schedule, () => runDigest());
  },
  stop() { /* cron.destroy() */ }
});
```
- `node-cron` npm: standard Node.js cron library
- Plugin starts its own background service with internal cron

**Option B — Gateway cron.add RPC**
- Plugin registers a Gateway RPC method that the cron system can call
- More complex, less self-contained

**Option C — Manual trigger only (skip auto-schedule)**
- Plugin exposes a `rss_run_digest` agent tool
- Emre or the heartbeat calls it manually
- Simplest for v0.1

### Recommendation
**v0.1:** Option C (manual tool trigger) + Option A (node-cron) configurable.
Default: schedule via `node-cron` in `registerService`. If `config.schedule` is empty → manual only.

---

## Summary + Recommendations for Opus

| Decision                     | Recommendation                                      |
| ---------------------------- | --------------------------------------------------- |
| RSS parsing                  | `rss-parser` npm — TypeScript native, no brainer    |
| CVE enrichment               | NVD API v2.0 via fetch, optional `nvdApiKey` config |
| Ghost auth                   | `jsonwebtoken` npm, HS256, regenerate per call      |
| Scheduling                   | `node-cron` in `api.registerService()` + manual tool |
| Python subprocess?           | NO — pure TypeScript; all deps available as npm     |
| Dependencies (total)         | `rss-parser`, `jsonwebtoken`, `node-cron`           |

### Open decisions for Opus
1. Notification: use OpenClaw CLI subprocess (`openclaw message send`) or plugin internal RPC?
2. Config: should `nvdApiKey` be optional? (yes, rate limit is fine for monthly runs)
3. Error handling strategy: fail silently and log, or throw and notify?
4. Multi-feed: run all feeds in parallel or sequential? (NVD rate limit suggests sequential)
5. Ghost URL: should it be in config or read from env var (GHOST_API_URL)?
