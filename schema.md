# SignalOS `ads.json` schema

`ads.json` is the single source of truth (same pattern as `today.json` in daily-bullet-journal). Each entry is one tracked ad.

```json
{
  "ad_id": "meta_120211XXXXXXXX",
  "advertiser": "Enneagram MBA",
  "category": "enneagram_at_work",
  "platform": "meta",
  "creative_type": "video",
  "creative_url": "https://www.facebook.com/ads/library/?id=...",
  "transcript": "Full transcribed audio of the ad, plus any on-screen text callouts noted separately below.",
  "on_screen_text": ["Text overlay 1", "Text overlay 2"],
  "ad_type_tag": "testimonial",
  "hook": "Short description of the opening line/frame and why it stops the scroll",
  "offer": "What's being offered — free workshop, paid kit, discovery call, etc.",
  "cta": "Book Now / Learn More / Download Kit",
  "first_seen": "2026-09-15",
  "last_seen": "2026-09-22",
  "days_running": 7,
  "status": "active",
  "hook_pattern": "success_paradox",
  "signal": {
    "long_runner": false,
    "iterated": false
  }
}
```

## Field notes

- **`ad_type_tag`** — one of your 7 types: `ugc_talking_head`, `static`, `features`, `offer`, `educational`, `testimonial`, `time_sensitive`. This is the field that makes the swipe file actually usable — everything else is metadata, this is the taxonomy.
- **`status`** — `active` while the ad still appears in the Ad Library query; flips to `inactive` the first refresh it no longer shows up. An ad going `inactive` after a short `days_running` is itself a signal (they tested it and killed it).
- **`category`** — must match a key in `competitors.json` (`enneagram_at_work`, `executive_coaching`, `fractional_coo`), so the frontend can group by market.
- **`creative_type`** — `video` or `static`. Static ads skip transcription and go straight to the tagging prompt against the image + copy.
- **`hook_pattern`** — short free-text label for the rhetorical hook shape (e.g. `success_paradox`, `stat_pattern_interrupt`, `identity_callout`), assigned during the weekly insights pass (see `insights-prompt.md`), not at initial tagging. Null until an ad has been through that pass at least once. This is what lets the insights step group ads by pattern across competitors instead of just by advertiser.
- **`signal`** — set by the weekly insights pass, not the tagging prompt. `long_runner`: true once `days_running >= 21` while still `active`. `iterated`: true if this advertiser has 2+ simultaneously-active ads sharing this hook/offer. Both are proxies for "this is probably working" — Meta doesn't expose real spend/CTR data outside the advertiser's own account, so longevity and repetition are the only signal available. Never treat these as real performance metrics in anything downstream.

## Top-level file shape

```json
{
  "last_refreshed": "2026-09-22T13:15:00Z",
  "ads": [ /* array of records above */ ]
}
```
