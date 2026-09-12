# SignalOS

Competitor ad signal + recommendation platform for the StackOS/NimblyStack product line, tracking Todd's own market first (Enneagram at Work competitors, executive coaches, fractional COOs).

- `competitors.json` -- tracked advertisers by category
- `ads.json` -- single source of truth for tracked ads (GitHub Contents API: GET for SHA + content, PUT with new content + that SHA -- same pattern as `zakiwarfel/daily-bullet-journal`)
- `schema.md` -- the `ads.json` record schema, including the 7-type ad taxonomy and the `signal`/`hook_pattern` fields used for pattern-finding
- `tagging-prompt.md` -- per-ad Claude tagging prompt, run during ingestion
- `cowork-ingestion-prompt.md` -- the weekly scheduled job: browses the public Meta Ad Library per advertiser (no API, no credentials, no Page IDs), diffs against `ads.json`, tags new ads, writes back
- `insights-prompt.md` -- chained after ingestion in the same weekly job: scores signal strength (longevity + iteration, the only proxy available since Meta exposes no real performance data outside the advertiser's own account), finds cross-competitor patterns, writes `recommendations.md`
- `ad-build-prompt.md` -- on-demand only, not scheduled: turns a chosen recommendation into an actual draft ad script/copy in Todd's voice
- `recommendations.md` -- current signal + recommendations, regenerated weekly

GitHub Pages frontend deferred until the pipeline has run for real.
