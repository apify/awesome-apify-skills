# Cost guardrails and error recovery

## Live prices, not remembered ones

```bash
apify actors info "johnvc/isolved-jobs-api" --json \
  --user-agent apify-awesome-skills/apify-isolved-jobs-api \
  2>/dev/null
```

Read `pricingInfos` from the output. Events are per delivered row; filtered rows are never charged, and there is no start fee.

## Guardrails

- Cap every exploratory run: `maxJobs: 25` (jobs) or `maxTenants: 25` (discovery) until the row shape is confirmed.
- Filters run before billing. Push `titleKeywords`, `locationKeywords`, `employmentType`, `hasSalary`, and `updatedAfter` into the input rather than filtering downstream.
- Description formats are per-row add-on events. Metadata-only rows (all description toggles off) are the cheapest job rows.
- The whole-run `report` is one flat event; skip it in pipelines.
- `urlsOnly` mode is the cheapest full index of a tenant: links and change timestamps, no descriptions.

## Error recovery

- `board_not_found`: the tenant has no public job sitemap; it may not exist or has no live jobs. Paste the tenant URL when a slug will not resolve.
- `http_error`: transient upstream answer; the row says the HTTP status. Retry once, then check the tenant site in a browser.
- `invalid_url`: the entry is not an isolved tenant slug, tenant URL, or job URL.
- Errors are in-band dataset rows with `resultType: "error"`; pipelines should branch on `resultType`, not on run status.

## Freshness facts worth knowing

- isolved publishes a per-job last-modified stamp in the sitemap, so `updatedAfter` is a real new-or-changed cutoff, not just a new-postings one. It is date-granular, so use day-sized windows like `25h`.
- The bundled tenant directory is rebuilt from isolved's public sitemap index, which lists every tenant, so discovery re-enumerates the live universe rather than a frozen snapshot.
