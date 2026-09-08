# Cost guardrails and error recovery

## Cost guardrails

- `maxJobs` is the primary spend cap. Start at 25 for a first look, then raise it once the row shape is confirmed.
- The base `job-result` event carries the source-HTML description and salary. The `job-description-markdown` and `job-description-text` add-ons bill only on rows that carry them, so leave them off unless a pipeline needs converted text.
- Filters (`titleKeywords`, `departments`, `locationKeywords`, `remoteOnly`, `employmentTypes`, `newerThan`) run before billing. Filtered jobs and expired postings cost nothing.
- Confirm live prices before a wide crawl:

```bash
apify actors info "johnvc/paylocity-jobs-api" --json \
  --user-agent apify-awesome-skills/apify-paylocity-jobs-api \
  2>/dev/null
```

- Rough confirmation thresholds: mention the estimate under about $5, warn over about $5, get explicit confirmation over about $20. Present cost as "around $X", never a guarantee.

## Error recovery

- `error` rows are in-band, so a pipeline sees them without reading logs. Each carries a sanitized `errorMessage` and its context. They are never charged.
- A board GUID that no longer resolves reports that it is not public. Confirm the GUID from the live `recruiting.paylocity.com` careers page.
- A company name that does not resolve: pass the board GUID or the full board URL instead of the name.
- Zero rows and no error row means your filters removed everything. Relax `newerThan` or `titleKeywords`.
- Dedupe across runs on `url`, never on title; a company can edit a title without changing the posting.
