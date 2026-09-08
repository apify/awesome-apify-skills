# Cost guardrails and error recovery: Clutch company data

## Confirm live prices before a batch

Prices can change. Read them from the live Actor rather than trusting a copied number:

```bash
apify actors info "johnvc/clutch-agency-api" --json \
  --user-agent apify-awesome-skills/apify-company-data-api \
  2>/dev/null
```

## Cost model

- Pay per event, no start fee. A run that delivers nothing costs almost nothing.
- `profile-scraped`: one per company record delivered.
- `review-scraped`: one per verified review row, only when `includeReviews` is true.
- Markdown output is free; `html` output costs a second request per profile.
- `maxItems` caps the whole run (profiles plus reviews). Set it before a wide batch.

Rule of thumb for confirmation: mention cost under about $5, warn over about $5, get explicit sign-off over about $20. Always phrase as "around $X".

## Guardrails

1. Start with one or two profiles and `includeReviews:false`. Inspect the shape before paying for a wide batch.
2. Reviews multiply rows fast. A company with 175 reviews and no cap is 175 review rows plus one profile row. Set `maxReviewsPerProfile`.
3. De-duplicate your input list; the Actor also de-dupes so one company is billed once, but a clean input keeps the run legible.
4. Keep `html` off unless you truly need the raw page. It doubles the per-profile request cost for data you usually already have as fields.

## Error recovery

- `result_type: "error"` rows carry `error_message` and `error_type`. A bad slug or a removed profile lands here rather than aborting the run.
- Zero rows: check for error rows first. If there are none and you asked for profiles, your `profileUrls` list was empty after normalization (for example a non-Clutch URL).
- A short profile is not an error. Many Clutch pages are genuinely thin.
- Retry a transient failure once. Persistent failures on one slug usually mean the profile no longer exists; drop it.
