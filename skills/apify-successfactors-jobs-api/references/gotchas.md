# Gotchas: cost guardrails and error recovery

## Read the live price before a big run
```bash
apify actors info "johnvc/sap-successfactors-jobs-api" --json \
  --user-agent apify-awesome-skills/apify-successfactors-jobs-api 2>/dev/null
```
Look at the pay-per-event events; the base job plus per-format and detail add-ons.

## Cost control
- Billing is per delivered row. Filters (title, jobFunction, location, remote, activeOnly) run before billing, so filtered rows cost nothing.
- The base job record is one event. Add-ons (descriptionMarkdown, descriptionHtml, descriptionText, detail fields, run report) bill only on rows that carry them.
- Caps: `maxJobs` (whole run), `maxJobsPerTenant`, `maxTenants`. Start with `maxJobs` of 10 to 25 while shaping filters.
- `includeDetailFields` adds one request per job; leave it off unless you need the posted date, department, facility, shift type, or travel.

## Error rows (resultType=error)
- `tenant_not_found`: the company did not resolve to a live feed. Pass the career-site URL and check `didYouMean`.
- `job_not_found`: a single-job URL pointed at a closed or expired posting.
- `http_error`: the career site answered abnormally; retry later.

## Empty results
- Filters may be too tight, or `activeOnly` dropped expired postings. Widen the filters or set `activeOnly` to false.
- A bare company name may not be in the bundled directory; pass the career-site host or URL directly.
