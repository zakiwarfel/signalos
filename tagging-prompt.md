# SignalOS tagging prompt

Used once per new ad found during ingestion. Fill in the bracketed fields and send to Claude.

---

You're analyzing a competitor ad for a swipe file. Classify it using this 7-type framework:

1. **UGC/Talking-Head** — a person speaking directly to camera, creator-style
2. **Static Image** — single image or graphic, no video
3. **Features Ad** — highlights specific product/service features or mechanics
4. **Offer Ad** — leads with a specific deal, price, or bundle
5. **Educational Ad** — teaches something, positions the advertiser as the expert
6. **Testimonial Ad** — a customer/client telling their own result or story
7. **Time-Sensitive Ad** — urgency or scarcity framing (deadline, limited spots, closing soon)

**Advertiser:** [advertiser name]
**Category:** [enneagram_at_work / executive_coaching / fractional_coo]
**Transcript:** [full transcript, or "N/A — static ad" if none]
**On-screen text:** [list of text overlays, or "N/A"]
**Days running so far:** [number, if known]

Return only this JSON, no other text:

```json
{
  "ad_type_tag": "one of: ugc_talking_head, static, features, offer, educational, testimonial, time_sensitive",
  "hook": "one sentence describing the opening line/frame and why it stops the scroll",
  "offer": "what's actually being offered",
  "cta": "the literal call-to-action text or button",
  "notes": "anything unusual or worth flagging — e.g. reused hook from a prior ad, notable angle shift"
}
```

If an ad genuinely straddles two types (e.g. a testimonial delivered as a talking-head video), pick the type that describes the *format* (talking-head) and note the secondary element (testimonial content) in `notes` — keeps the tag field clean for filtering later.
