# Cost guardrails and error recovery

## Live prices, not remembered ones

```bash
apify actors info "johnvc/isolved-jobs-api" --json \
  --user-agent apify-awesome-skills/apify-companies-using-isolved \
  2>/dev/null
```

Read `pricingInfos` from the output. The `tenant-row` event is per live-verified tenant; dead tenants are skipped and never charged, and there is no start fee.

## Guardrails

- Cap every exploratory sweep: `maxTenants: 25` until the row shape is confirmed.
- `discoveryQuery` narrows the sweep before billing; use it instead of pulling the whole directory and filtering downstream.
- `verifyTenants: false` lists the directory snapshot without live probes, faster and cheaper, at the cost of a live `jobCount`.
- Dead tenants (no public sitemap) are skipped silently and never billed, so a stale prospect list costs only for the ones that resolve.

## Error recovery

- `board_not_found` on a prospect check: that tenant has no public job sitemap; confirm the slug or paste the tenant URL.
- `http_error`: transient upstream answer; the row says the HTTP status. Retry once, then check the tenant site in a browser.
- `invalid_url`: the entry is not an isolved tenant slug, tenant URL, or job URL.
- Errors are in-band dataset rows with `resultType: "error"`; pipelines should branch on `resultType`, not on run status.

## Freshness facts worth knowing

- The bundled tenant directory is rebuilt from isolved's public sitemap index, which lists every tenant, so a sweep re-enumerates the live universe rather than a frozen snapshot. New employers appear on their own.
- `jobCount` is the live count at run time and moves as employers post and close roles.
