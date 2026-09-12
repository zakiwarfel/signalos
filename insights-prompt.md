# SignalOS weekly insights & recommendations — CoWork prompt

Runs immediately after the ingestion step (`cowork-ingestion-prompt.md`), as part of the same weekly job, using the just-updated `ads.json`.

## What this step is for

The raw tracker (who's running what) isn't the deliverable — Todd wants a signal + recommendation layer: which ads are actually working, what pattern that reveals, and a concrete ad concept to consider building. This step turns `ads.json` into `recommendations.md`.

**Honest constraint:** Meta does not expose spend, CTR, or conversion data for non-political ads to anyone but the advertiser — not through the API, not through the public library. There is no real performance number available here. What's used instead is the same proxy every professional ad-research tool relies on: advertisers kill underperforming ads fast and keep running (or duplicate) winners. Longevity and iteration are the signal. Never describe a flagged ad as "high-performing" or imply real metrics back it — say "long-running" / "actively iterated," which is what's actually known.

## Step 1: Score each active ad's signal

- **Long runner:** `days_running >= 21` on an ad still `active`.
- **Iterated:** the same advertiser has 2+ simultaneously-active ads sharing the same core hook or offer (Meta often serves the same creative under multiple Library IDs, or an advertiser tests close variants at once — both count).

## Step 2: Find cross-competitor patterns

Group active ads — especially long-runner/iterated ones — by hook shape, offer structure, and CTA friction level, across ALL advertisers and categories together, not per-competitor. Look specifically for:

- Repeated hook shapes (paradox/reframe, stat-led pattern interrupt, identity callout, "you're doing X and it's not working")
- Repeated CTA friction level (book-a-call vs. quiz vs. free guide vs. free training vs. low-price tripwire)
- Repeated offer structure (certification/practitioner model, done-for-you service, self-serve tool, info-product funnel)

**Confidence matters — state it explicitly.** A pattern showing up across 2+ unrelated advertisers is market-validated. A pattern in only one advertiser's ad is that company's own bet, not a market signal. Never blur the two.

## Step 3: Write `recommendations.md`

Same repo, same GitHub Contents API mechanics as `ads.json` (GET for SHA + content, PUT with new content + that SHA). Structure:

- **What's working right now** — 2-4 patterns, each naming the advertisers/categories behind it and its confidence level (cross-validated vs. single-advertiser bet)
- **New this week** — diff against the previous `recommendations.md`: what pattern is newly emerging, what previously-flagged long-runner went inactive (a pattern fading is itself worth noting, not just silently dropped)
- **Suggested ad concepts** — 1-3 concrete concepts. Each names: which of Todd's 7 ad types, a one-line premise, which pattern(s) it draws on, and which business it fits (Enneagram at Work vs. StackOS/NimblyStack — check `competitors.json` category and Todd's own positioning notes)

## Step 4: Stop at the concept level

Do not auto-generate ad scripts or copy in this step. Turning a chosen concept into an actual draft is a separate, on-demand step (`ad-build-prompt.md`) — Todd picks which concept to run with and needs to be involved (he's on camera for UGC ads and has to choose which real client story/result to use for testimonials).

## End-of-run summary addition

Alongside the ingestion summary, add: any new long-runner/iterated ads this cycle, and a one-line pointer to what changed in `recommendations.md`.
