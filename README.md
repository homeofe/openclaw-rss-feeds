# @elvatis/openclaw-rss-feeds

OpenClaw plugin that turns any RSS/Atom feed into an automated monthly (or custom-schedule) digest — with CVE enrichment, Ghost CMS draft creation, and channel notifications.

## What it does

Fetches and filters RSS/Atom feeds on a configurable schedule, enriches security-related content with NVD CVE data, generates a formatted HTML digest as a Ghost CMS draft, and notifies configured channels (WhatsApp, Telegram, etc.).

## Use cases

- **Security vendor feeds** (Fortinet, Cisco, Palo Alto, Microsoft, etc.) with CVE enrichment
- **Product release feeds** (firmware updates, changelogs, patch notes)
- **News digest** (any RSS feed filtered by keyword)
- **Blog content pipeline** (auto-draft from curated feeds)

## Architecture

```
Schedule (cron)
  └── @elvatis/openclaw-rss-feeds plugin
        ├── Fetch RSS/Atom feeds
        ├── Filter by keywords + date range
        ├── Enrich with NVD CVE data (optional)
        ├── Format as HTML digest
        ├── Create Ghost CMS draft (optional)
        └── Notify configured channels
```

## Installation

```bash
openclaw plugins install @elvatis/openclaw-rss-feeds
```

## Configuration

```json
{
  "plugins": {
    "entries": {
      "openclaw-rss-feeds": {
        "config": {
          "feeds": [
            {
              "id": "fortinet",
              "name": "Fortinet Firmware",
              "url": "https://support.fortinet.com/rss/firmware.xml",
              "keywords": ["fortinet"],
              "enrichCve": true,
              "cvssThreshold": 6.5,
              "docsUrlTemplate": "https://docs.fortinet.com/document/{product}/{version}/release-notes",
              "productHighlightPattern": "Forti(?!net)[a-zA-Z0-9]+(?:\\s+Cloud)?"
            },
            {
              "id": "cisco",
              "name": "Cisco Security Advisories",
              "url": "https://tools.cisco.com/security/center/psirtrss20/CiscoSecurityAdvisory.xml",
              "keywords": ["cisco"],
              "enrichCve": true,
              "cvssThreshold": 7.0,
              "productHighlightPattern": "Cisco\\s+[A-Z][a-zA-Z0-9]+"
            },
            {
              "id": "paloalto",
              "name": "Palo Alto Networks",
              "url": "https://security.paloaltonetworks.com/rss.xml",
              "keywords": ["palo alto", "pan-os"],
              "enrichCve": true,
              "cvssThreshold": 7.0,
              "docsUrlTemplate": "https://docs.paloaltonetworks.com/pan-os/{version}/release-notes"
            }
          ],
          "schedule": "0 9 1 * *",
          "ghost": {
            "url": "https://your-ghost-instance.com",
            "adminKey": "id:secret"
          },
          "notify": ["whatsapp:+49xxxxxxxxxx"]
        }
      }
    }
  }
}
```

## Feed Config Fields

| Field | Type | Description |
|---|---|---|
| `id` | `string` | Unique feed identifier |
| `name` | `string` | Human-readable feed name |
| `url` | `string` | RSS/Atom feed URL |
| `keywords` | `string[]` | Optional keyword filter (title + content, case-insensitive) |
| `enrichCve` | `boolean` | Enable NVD CVE enrichment for this feed |
| `cvssThreshold` | `number` | Minimum CVSS score to include (default 6.5) |
| `tags` | `string[]` | Ghost tags for the generated draft |
| `docsUrlTemplate` | `string` | URL template for documentation links. Use `{product}` and `{version}` as placeholders. Example: `"https://docs.example.com/{product}/{version}/release-notes"` |
| `productHighlightPattern` | `string` | Regex pattern to highlight product names in CVE descriptions. Example: `"Forti(?!net)[a-zA-Z0-9]+"` |

## Status

Work in progress. See `.ai/handoff/STATUS.md` for current build state.
