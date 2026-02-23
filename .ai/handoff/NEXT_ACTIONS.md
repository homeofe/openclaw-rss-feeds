# openclaw-rss-feeds — Next Actions

## Current State

P4 implementation work is complete in repository (`main`) and pushed.

## Remaining Steps

1. **Manual release prep**
   - Bump version if needed
   - Run final `npm pack` sanity check

2. **Manual publish (human-triggered)**
   - `npm publish` for `@elvatis/openclaw-rss-feeds`
   - Not executed by agent on purpose

3. **Community submission**
   - Submit plugin to OpenClaw community catalog
   - Include README and config example from repository

4. **Optional hardening follow-up**
   - Add retry/backoff option for RSS fetches
   - Add integration test for full `rss_run_digest` flow with mocked dependencies
