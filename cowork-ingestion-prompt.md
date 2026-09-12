# SignalOS weekly ingestion — CoWork scheduled prompt

Run weekly (suggest Monday morning, ahead of any content planning for the week).

**Architecture note (updated 2026-09-12):** this runs entirely through Claude's browser tools against the public Ad Library website (facebook.com/ads/library) — no Meta Developer account, no app, no access token, no Page ID lookup. The official Ad Library *API* was evaluated and rejected for this use case: it only returns non-political commercial ads for advertisers with EU/UK ad reach (a US-only advertiser is invisible to it), and even a basic read query requires completing Meta's political-advertiser identity-verification flow — which would mean every future customer of this as a StackOS expansion pack having to falsely attest they intend to run political/social-issue ads just to unlock ad research. The public website has no such gates: anyone can browse any advertiser's active ads, any country, no login. This keeps the product credential-free and portable.

## Steps

1. **Fetch current state.** GET `https://api.github.com/repos/[your-account]/signalos/contents/ads.json` with `Authorization: Bearer [GitHub PAT]`. Decode base64, parse JSON, keep the SHA for the later write.

2. **Fetch tracked advertisers.** GET the same repo's `competitors.json`, decode, and build a flat list of `{name, website, category}` across all three categories.

3. **For each advertiser, browse the Ad Library.** Open `https://www.facebook.com/ads/library/?active_status=active&ad_type=all&country=ALL&q=<name>&media_type=all` and read the page text. Keyword search can surface false positives (an unrelated ad that merely mentions the search term) — only keep results whose displayed advertiser/Page name actually matches the tracked advertiser (use the `website` field to disambiguate if two advertisers share a name). If an advertiser has zero active ads, note that in the summary and move on — it's not an error.

   **Known limitation, confirmed by live testing (2026-09-12):** Meta's Ad Library search does token/keyword matching, not exact-phrase matching — quoting the query does not help. A distinctive name (e.g. "Vanguard XXI") returns a small, clean result set. A name built from common words (e.g. "The COO Solution") returns tens of thousands of unrelated global results, because it matches "COO" and "Solution" as independent substrings anywhere in the ad library. When a search for a tracked advertiser returns an unexpectedly huge result count (roughly >500), don't trust position in the list — either (a) scan specifically for the exact advertiser/Page name as it appears in `competitors.json` and cross-check the linked website against the `website` field, or (b) if that's not findable in a reasonable number of results, flag the advertiser as "name too generic to search reliably" in the end-of-run summary rather than silently skipping or guessing. Don't burn excessive time paging through a 30,000+ result set.

4. **Extract each matching ad's data** directly from the page: Library ID, "Started running on" date, platforms shown, and the full ad copy/body text (the public library shows this inline for both video and static ads, plus on-screen text callouts where shown). No separate transcription step is needed unless the ad is pure video with no caption/text shown on the library page — in that case note `transcript: "not available from library page"` rather than guessing.

5. **Diff against `ads.json`.**
   - Any Library ID found that isn't already in `ads.json` → new ad, needs tagging (step 6).
   - Any `ad_id` in `ads.json` marked `active` that no longer appears for that advertiser → flip `status` to `inactive`, set `last_seen` to today.
   - Any ad still found and still in `ads.json` → update `last_seen` and `days_running`, no re-tagging needed.

6. **For each new ad:** run the tagging-prompt.md prompt against the extracted copy (fill in advertiser/category/text) to get `ad_type_tag`, `hook`, `offer`, `cta`, `notes`. Assemble the full record per schema.md — set `ad_id` to `meta_<library_id>` and `creative_url` to `https://www.facebook.com/ads/library/?id=<library_id>`.

7. **Write back.** PUT to the same `ads.json` contents endpoint with the updated array, the original SHA, and `last_refreshed` set to now (base64-encode carefully for any special characters in ad copy).

8. **End-of-run summary** (for CoWork to report back, not to write to the repo): count of new ads found, count flipped to inactive, any advertisers with zero active ads this cycle.

9. **Chain into insights.** Once `ads.json` is written back, run `insights-prompt.md` against the freshly updated file in the same job — it scores signal strength (long-runner / iterated), finds cross-competitor patterns, and writes `recommendations.md`. This is what turns the raw tracker into something worth reading each week, not just a data dump. Do not run `ad-build-prompt.md` here — that stays on-demand, triggered by Todd picking a concept.

## Notes carried over from the bullet-journal build

- The SHA must be included on the PUT or it fails — same as `today.json`.
- Base64-encode carefully — ad copy will have more punctuation/unicode edge cases than calendar events did.
- Keep the GitHub PAT stored in CoWork, same as the daily bullet journal job. No other credentials are needed for this version.
