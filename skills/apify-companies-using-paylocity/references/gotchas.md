# Cost guardrails and error recovery

## Cost guardrails

- `maxCompanies` is the primary spend cap for a discovery run. Start at 25, then raise it once the row shape is confirmed.
- Each live or empty `company-result` row is one cheap event; unverified, dead, and error rows are delivered free, so a `verifyLive: false` first pass is nearly free.
- `crawlDepth` widens web-archive coverage but makes the sweep slower; raise it only when you need broader discovery.
- Confirm live prices before a wide sweep:

```bash
apify actors info "johnvc/paylocity-jobs-api" --json \
  --user-agent apify-awesome-skills/apify-companies-using-paylocity \
  2>/dev/null
```

- Rough confirmation thresholds: mention the estimate under about $5, warn over about $5, get explicit confirmation over about $20. Present cost as "around $X", never a guarantee.

## Error recovery

- `error` rows are in-band and never charged; each carries a sanitized `errorMessage`.
- Few rows on a narrow `discoveryQuery`: broaden the term or raise `crawlDepth`.
- Many `unverified` rows: turn `verifyLive` on for a real `jobCount` and `status`.
- A discovered board with no jobs later carried `status: empty`; the company has no open roles right now.
- Dedupe across sweeps on `boardGuid`, never on company name.
