---
name: apify-isolved-jobs-api
description: "Scrape isolved jobs from any isolvedhire.com career site with the isolved Jobs API Actor (johnvc/isolved-jobs-api). Returns live job rows as JSON: title, hiring organization, tenant, employment type, locations with derived country, posted date, a sitemap last-modified stamp, apply URL, the employer's published salary when present, and the description as Markdown by default. Takes tenant slugs, tenant URLs, or single job URLs, and resolves legacy ApplicantPro sites too; filters by title, location, employment type, and salary before billing. updatedAfter turns a daily schedule into a real new-or-changed-postings feed using each job's own lastmod, checked before any page is fetched, so you pay only for what moved. Use when someone wants isolved jobs data, an isolved jobs api, to scrape isolved or ApplicantPro job postings for a job board or aggregator, isolved ATS listings, or change detection on isolved postings. Billed per job delivered with no start fee, and MCP-ready for Claude and other AI agents."
author: John Cole
author_url: https://github.com/johnisanerd
license: MIT
metadata:
  version: "1.0"
---

# isolved Jobs, as Rows You Can Query

Tenant slugs or job URLs in, live isolved jobs out as structured rows, each carrying the posting's own last-modified stamp so a scheduled run can return only what changed.

## When to use this skill

- Someone wants isolved jobs and does not want to read isolvedhire.com career sites by hand.
- You are filling a job board, a market-intel dashboard, or a sourcing tool with roles from isolved-hosted and ApplicantPro career pages.
- You need a real change feed: isolved publishes a per-job last-modified stamp, so a daily run can return only new or edited postings with no state to keep.
- You went looking for an isolved jobs API and found the vendor's product only, not a way to read the public postings as data.

Not for: finding which employers run isolved in the first place. Use the companion `apify-companies-using-isolved` skill, built on the same Actor but shaped for tenant discovery. See `references/actor-index.md`.

## What you get

One dataset row per job. `resultType` separates `job` rows from `error` rows.

Core fields:

- `id`, `url` (canonical, safe as a dedupe key), `applyUrl`, `title`
- `organization`, `organizationUrl`, `tenant`, `tenantUrl`
- `employmentType`, `locationsDerived`, `isRemote`
- `datePosted`, `dateUpdated` (the sitemap last-modified stamp; date-granular)
- `salaryRaw` (the employer's published pay, verbatim; usually null on isolved)
- `descriptionMarkdown` (default add-on), `descriptionHtml`, `descriptionText` (opt-in add-ons)
- `source` (always `isolved`), `sourceType` (`ats`), `sourceUrl`, `scrapedAt`

The Actor ships dataset views for the console: `overview`, `changes` (ordered by last-updated for monitoring), and `tenants`.

## Prerequisites

- Apify account (sign up at https://apify.com?fpr=9n7kx3&fp_sid=awesomeskills).
- Authentication via `apify login`, or an `APIFY_TOKEN` environment variable (Apify Console, Settings, Integrations).

## The Actor

- Store page: https://apify.com/johnvc/isolved-jobs-api?fpr=9n7kx3&fp_sid=awesomeskills
- Actor ID: `johnvc/isolved-jobs-api`
- Pricing: pay per event, no start fee. See the cost section below and `references/gotchas.md` for the live-price command.

## Run it with the Apify CLI

One tenant, full job rows, capped at 25:

```bash
apify actors call "johnvc/isolved-jobs-api" -i '{"tenants":["isolved"],"maxJobs":25}' \
  --json \
  --user-agent apify-awesome-skills/apify-isolved-jobs-api \
  2>/dev/null
```

Only postings that changed in the last day, the shape to put on a daily schedule:

```bash
apify actors call "johnvc/isolved-jobs-api" -i '{"tenants":["isolved","davidsonoil"],"updatedAfter":"25h","maxJobs":200}' \
  --json \
  --user-agent apify-awesome-skills/apify-isolved-jobs-api \
  2>/dev/null
```

Full-time roles only, descriptions off for cheaper metadata rows:

```bash
apify actors call "johnvc/isolved-jobs-api" -i '{"tenants":["isolved"],"employmentType":["FULL_TIME"],"includeDescriptionMarkdown":false,"maxJobs":100}' \
  --json \
  --user-agent apify-awesome-skills/apify-isolved-jobs-api \
  2>/dev/null
```

A legacy ApplicantPro site resolves automatically:

```bash
apify actors call "johnvc/isolved-jobs-api" -i '{"tenants":["https://someco.applicantpro.com"],"maxJobs":25}' \
  --json \
  --user-agent apify-awesome-skills/apify-isolved-jobs-api \
  2>/dev/null
```

Confirm the live schema and prices before a large batch:

```bash
apify actors info "johnvc/isolved-jobs-api" --json \
  --user-agent apify-awesome-skills/apify-isolved-jobs-api \
  2>/dev/null
```

Read the rows back from a finished run:

```bash
apify datasets get-items <DATASET_ID> --format json \
  --user-agent apify-awesome-skills/apify-isolved-jobs-api \
  2>/dev/null
```

Every call carries the three flags this repo expects: `--json` (or `--format json`), `--user-agent apify-awesome-skills/apify-isolved-jobs-api`, and `2>/dev/null`.

## Run it from Claude or another AI agent (MCP)

The Actor is MCP-ready. Add the hosted server URL:

`https://mcp.apify.com/?tools=actors,docs,johnvc/isolved-jobs-api`

Then ask, for example: "Pull the newest isolved postings from these tenants in the last week and return title, location, and apply URL." MCP setup docs: https://docs.apify.com/platform/integrations/mcp

## Workflow

1. Start with one tenant and `maxJobs` around 25. Look at the row shape before you pay for a wide crawl. A tenant slug is the part before `.isolvedhire.com`.
2. Tenant slugs, full tenant URLs, and single job URLs all work as input, mixed freely. Legacy `applicantpro.com` URLs resolve too.
3. Filter at the source, not downstream. `titleKeywords`, `locationKeywords`, `employmentType`, `hasSalary`, and `updatedAfter` all drop jobs before they are billed.
4. Use `updatedAfter` for a change feed and `publishedAfter` for a genuinely-new-roles feed. `updatedAfter` is checked against the sitemap stamp before any page is fetched, so unchanged postings are never even downloaded.
5. Keep `includeDescriptionMarkdown` on for LLM pipelines; turn it off for metadata-only rows at a lower per-row cost.
6. Check `resultType` before treating a row as a job. An `error` row carries `errorCode` and a human-readable `errorMessage`.
7. Dedupe downstream on `url`. It is canonical and survives an employer editing the title.

## Inputs

- `tenants` (array): tenant slugs, tenant URLs, or single job URLs, mixed freely. Empty sweeps the bundled directory.
- `startUrls` (array): the same values in URL-list form; merged with `tenants`.
- `outputMode` (enum `jobs`, `urlsOnly`, `tenantsOnly`, default `jobs`)
- `titleKeywords`, `locationKeywords` (arrays): keep-only filters, run before billing.
- `employmentType` (array): schema.org codes like `FULL_TIME`, `PART_TIME`; plain wording is accepted.
- `hasSalary` (boolean, default false): keep only postings with a published salary.
- `updatedAfter` (string): `24h`, `7d`, `2w`, or an ISO date, against the sitemap last-modified stamp.
- `publishedAfter` (string): same grammar, against the posting's `datePosted`.
- `includeDescriptionMarkdown` (boolean, default true), `includeDescriptionHtml`, `includeDescriptionText` (default false)
- `report` (enum `none`, `markdown`, `html`): a whole-run digest in the key-value store.
- `maxJobs` (integer, default 100): the primary spend cap. `maxJobsPerTenant`, `maxTenants`, `maxConcurrency` refine it.
- `proxyConfiguration` (object): off by default; direct connections work.

## Cost

Billing is pay per event with no start fee, so a run that returns nothing costs almost nothing. Confirm live prices with the info command above rather than trusting a number copied here.

The base `job-result` event fires once per delivered job row. Add-on events fire only on rows that carry the extra: `job-description-markdown` (on by default), `job-description-html`, `job-description-text`, and a flat `run-report`. Tenant discovery rows and URL index rows have their own cheaper events.

Jobs removed by your filters are never charged. As a shape, 100 jobs with Markdown descriptions is small change.

Suggested confirmation thresholds: mention the estimate under about $5, warn the user over about $5, get explicit confirmation over about $20. Present cost as "around $X", never as a guarantee.

## Honest limits

- **Change detection is date-granular.** The sitemap last-modified stamp is a date, so `updatedAfter` resolves to days, not hours. A `25h` window on a daily schedule is the right shape.
- Salary appears only when the employer publishes structured pay on the posting, which most isolved employers do not, so `salaryRaw` is often null. Nothing is inferred by a model.
- Public career-site data only. No applicant data, no recruiter contacts, nothing behind a login.
- US-focused. isolved career sites are almost entirely United States employers.

## Troubleshooting

- `board_not_found`: the tenant has no public job sitemap; it may not exist or has no live jobs. Check the slug or paste the tenant URL.
- `http_error`: the source answered abnormally; the row says which HTTP status. Retry once before assuming anything.
- `invalid_url`: the entry is not an isolved tenant slug, tenant URL, or job URL.
- Zero rows and no error row: your filters removed everything. Relax `updatedAfter` or `titleKeywords` first.
- Duplicates across runs: dedupe on `url`, never on title.

See `references/gotchas.md` for cost guardrails and error recovery, and `references/actor-index.md` for the Actor routing table.

## Related Actors

- Greenhouse Job Board API: https://apify.com/johnvc/greenhouse-job-board-api?fpr=9n7kx3&fp_sid=awesomeskills
- Ashby Job Board API: https://apify.com/johnvc/ashby-job-board-scraper?fpr=9n7kx3&fp_sid=awesomeskills
- iCIMS Careers API: https://apify.com/johnvc/icims-careers-api?fpr=9n7kx3&fp_sid=awesomeskills
- Workday Careers API: https://apify.com/johnvc/workday-careers-api?fpr=9n7kx3&fp_sid=awesomeskills
