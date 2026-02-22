# P4 Review — Opus (Architektur)

**Reviewer:** Opus (Architect) | **Date:** 2026-02-22 | **Commit reviewed:** 42b2f04

## ADR-Konformität

- **D1 Pure TS:** ✅ Korrekt. Kein Python, kein Subprocess für Logik. Alle Deps sind npm-Pakete (rss-parser, jsonwebtoken, node-cron).
- **D2 Scheduling:** ✅ Korrekt. `node-cron` in `registerService` mit `start()`/`stop()`. `cron.validate()` wird geprüft. Manual trigger via `registerTool('rss_run_digest')`. Beides optional.
- **D3 Notification:** ✅ Korrekt. CLI-Subprocess via `openclaw message send`. Format `channel:target` wie im ADR.
- **D4 CVE optional:** ✅ Korrekt. Per-Feed via `enrichCve: true` + `keywords[]`. `nvdApiKey` optional. Ohne Key funktioniert es mit 6s Rate-Limit-Delay.
- **D5 Sequential:** ✅ Korrekt. Feeds werden sequentiell in `for...of` verarbeitet. 6s Sleep zwischen NVD-Requests.
- **D6 Config Schema:** ✅ Korrekt. `PluginConfig` Interface matcht das ADR-Schema exakt: `feeds[]`, `schedule?`, `lookbackDays?`, `ghost?{url, adminKey}`, `notify?[]`, `nvdApiKey?`.

## Plugin API Konformität

- `registerService({id, start, stop})` — ✅ matcht OpenClaw Plugin-Spec
- `registerTool({name, description, parameters, execute})` — ✅ matcht Plugin-Spec
- Default export `(api) => void` — ✅ korrekt
- `api.logger.info/warn/error` — ✅ konsistent genutzt (außer cveFetcher, siehe Minor)

## Gefundene Probleme

### Kritisch (gefixt in diesem Commit)

1. **Shell-Injection in notifier.ts** — `execSync()` mit String-Konkatenation und `shellEscape()` ist anfällig. Wenn ein `notify`-Target oder die Message-Payload Sonderzeichen enthält, könnte trotz Escaping ein Angriffsvektor bestehen (single-quote escaping hat Edge-Cases bei verschiedenen Shells).
   - **Fix:** Ersetzt durch `execFileSync()` (kein Shell-Subprocess, Argumente werden direkt als Array übergeben). Shell-Injection ist damit strukturell unmöglich.

2. **Ghost-Publish ohne äußeres try/catch** — `publishDraft()` hat zwar intern try/catch, aber wenn `makeGhostToken()` vor dem HTTP-Call wirft (z.B. ungültiges `adminKey`-Format), wurde der Fehler intern gefangen. Trotzdem: defensives Wrapping im Caller (`runDigest`) hinzugefügt, da Ghost-Failures laut ADR non-fatal sein sollen.
   - **Fix:** Äußeres try/catch um den gesamten Ghost-Block in `runDigest()`.

### Minor (nice-to-have, kein Blocker)

1. **cveFetcher nutzt `console.warn` statt `api.logger`** — Inkonsistent mit ADR-Punkt 5 (structured logging via `api.logger`). cveFetcher hat keinen Zugriff auf `api.logger`, weil es nur primitive Params bekommt. Empfehlung: Logger als optionalen Parameter durchreichen oder in v0.2 refactoren.

2. **`getFirmwareType()` Heuristik ist Fortinet-spezifisch** — Die Major/Feature/Patch-Erkennung basiert auf dem letzten Versionsteil (`0`=Major, `2`/`4`=Feature, rest=Patch). Das funktioniert für Fortinet, aber nicht generisch. Für v0.1 (Fortinet-only) akzeptabel.

3. **Kein `openclaw.plugin.json` Manifest** — Die Plugin-Spec erwartet ein Manifest mit `configSchema` und `uiHints`. `package.json` hat `openclaw.extensions`, aber das JSON-Schema für Config-Validierung fehlt. Sollte vor npm-Publish ergänzt werden.

4. **`dryRun`-Modus erstellt neues `PluginApi`-Objekt mit Spread** — Funktioniert, aber `api` könnte nicht-enumerable Properties haben. Sauberer wäre ein `skipPublish`-Flag im `runDigest`-Parameter.

5. **Kein Timeout/Retry bei RSS-Fetch** — `rss-parser` hat `timeout: 20000`, aber kein Retry. Bei flaky Feeds könnte ein einzelner Retry helfen.

## Empfehlung

**NEEDS_FIXES** → Kritische Fixes (Shell-Injection, Ghost try/catch) wurden direkt in diesem Commit behoben.

Nach den Fixes: **APPROVED für v0.1.0-beta**. Minor-Punkte als Issues tracken.
