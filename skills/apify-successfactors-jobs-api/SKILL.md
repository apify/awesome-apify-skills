---
name: apify-successfactors-jobs-api
description: "Pull live SAP SuccessFactors job postings from any public SuccessFactors career site with the Apify SAP SuccessFactors Jobs API Actor (johnvc/sap-successfactors-jobs-api). Returns job rows as JSON: title, company, employer, requisition ID, job function, location and country code, expiration date, remote flag, and the description as Markdown by default, with optional HTML or plain text. Turn on the detail add-on for the posted date, department, facility, shift type, and travel that only the job page carries. Takes a career-site host like jobs.sap.com, a full career-site URL, a single job URL, or a company name; filters by title, job function, location, remote, and active-only before billing. Use when someone wants successfactors jobs, a successfactors jobs api, a successfactors api without a key, to scrape SAP SuccessFactors career sites, or to feed a job board, aggregator, or sourcing tool with enterprise postings. Billed per job delivered with no start fee, and MCP-ready for Claude and other AI agents."
author: John Cole
author_url: https://github.com/johnisanerd
license: MIT
metadata:
  version: "1.0"
  keywords: ["successfactors", "successfactors jobs", "successfactors api", "ats", "jobs", "mcp"]
---

# SAP SuccessFactors Jobs, as Rows You Can Query

A career-site host, URL, or company name in, live SAP SuccessFactors jobs out as structured rows, read straight from the public feed each career site already publishes.

## When to use this skill

- Someone wants SuccessFactors jobs and does not want to read career sites like jobs.sap.com by hand.
- You are filling a job board, a market-intel dashboard, or a sourcing tool with roles from SuccessFactors-hosted careers pages.
- You went looking for a SuccessFactors API and found only the employer-side OData APIs you cannot sign up for.
- You need the whole career site in one pull, not page after page of search results.

Not for: using the data live from inside Claude or another MCP client. Use the companion `apify-successfactors-mcp` skill, built on the same Actor. See `references/actor-index.md`.

## What you get

One dataset row per job. `resultType` separates `job` rows from `error` rows.

Core fields:

- `requisitionId`, `url` (canonical, safe as a dedupe key), `applyUrl`, `title`
- `companyName`, `employer`, `tenant`, `careerSiteUrl`
- `jobFunction`, `location`, `countryCode`, `isRemote`
- `validThrough` (expiration date), and `datePosted` with the detail add-on
- `salaryDerived` plus flat `salaryMin`, `salaryMax`, `salaryCurrency`, `salaryPeriod` (parsed from the posting text when present)
- `descriptionMarkdown` (default add-on), `descriptionHtml`, `descriptionText` (opt-in add-ons)
- `source` (always `successfactors`), `sourceType`, `flavor`, `scrapedAt`

With `includeDetailFields` on, each row also carries `datePosted`, `department`, `facility`, `shiftType`, `travel`, and any employer `customFields`, fetched from the job page.

The Actor ships dataset views for the console: `overview`, `details`, and `tenants`.

## Prerequisites

- Apify account (sign up at https://apify.com?fpr=9n7kx3&fp_sid=awesomeskills).
- Authentication via `apify login`, or an `APIFY_TOKEN` environment variable (Apify Console, Settings, Integrations).

## The Actor

- Store page: https://apify.com/johnvc/sap-successfactors-jobs-api?fpr=9n7kx3&fp_sid=awesomeskills
- Actor ID: `johnvc/sap-successfactors-jobs-api`
- Pricing: pay per event, no start fee. See the cost section below and `references/gotchas.md` for the live-price command.

## Run it with the Apify CLI

Pull the first 100 live jobs from a career site, descriptions as Markdown:

```bash
apify actors call "johnvc/sap-successfactors-jobs-api" \
  -i '{"companies":["jobs.sap.com"],"maxJobs":100,"includeDescriptionMarkdown":true}' \
  --json \
  --user-agent apify-awesome-skills/apify-successfactors-jobs-api \
  2>/dev/null
```

Read a run's dataset by id:

```bash
apify datasets get-items <DATASET_ID> --format json \
  --user-agent apify-awesome-skills/apify-successfactors-jobs-api 2>/dev/null
```

Inspect the input schema:

```bash
apify actors info "johnvc/sap-successfactors-jobs-api" --input --json \
  --user-agent apify-awesome-skills/apify-successfactors-jobs-api 2>/dev/null
```

## Run it from Claude (MCP)

Add the hosted Apify MCP server and call the Actor as a tool:

```
https://mcp.apify.com/?tools=actors,docs,johnvc/sap-successfactors-jobs-api
```

Docs: https://docs.apify.com/platform/integrations/mcp

## Workflow

1. Identify the target career sites: a host such as `jobs.sap.com`, a full career-site URL, a single job URL, or a company name to match against the bundled directory.
2. Decide the output: `jobs` for full records, `urlsOnly` for the cheapest index, or `tenantsOnly` to list career sites.
3. Pick description formats (Markdown is on by default) and turn on `includeDetailFields` if you need the posted date or department.
4. Set filters (`titleKeywords`, `jobFunctions`, `locationKeywords`, `remoteOnly`, `activeOnly`) so filtered rows never bill.
5. Cap the run with `maxJobs` (and `maxJobsPerTenant`, `maxTenants`).
6. Call the Actor, then read the dataset. Dedupe on `url` or `requisitionId` across runs.

## Inputs

- `companies`: career-site hosts, full URLs, single-job URLs, or company names.
- `startUrls`: the same values as a URL list; merged with `companies`.
- `outputMode`: `jobs` (default), `urlsOnly`, or `tenantsOnly`.
- `titleKeywords`, `jobFunctions`, `locationKeywords`: keep-only filters, applied before billing.
- `activeOnly` (default true), `remoteOnly`: further pre-billing filters.
- `includeDescriptionMarkdown` (default true), `includeDescriptionHtml`, `includeDescriptionText`: per-format add-ons.
- `includeDetailFields`: posted date, department, facility, shift type, travel, custom fields (one extra request per job).
- `report`: write a Markdown or HTML run digest to the key-value store.
- `maxJobs`, `maxJobsPerTenant`, `maxTenants`, `maxConcurrency`: cost and scope caps.

## Cost guardrails

- Billing is per delivered row. Filters run before billing, so filtered jobs cost nothing.
- The base job record is one event; description formats, the detail fields, and the run report are separate add-ons billed only on rows that carry them.
- Start small: `maxJobs` of 10 to 25 while you shape filters, then raise it.
- Read the live price before a big run (see `references/gotchas.md`).

## Honest limits

- Salary is a best-effort regex over the posting text; many SuccessFactors postings publish no pay, so salary fields are often null.
- The public feed carries no posted date; `datePosted` comes only with the detail add-on.
- Coverage is the public career-site feed, so unlisted or gated postings are not returned.

## Troubleshooting

- An `error` row with `tenant_not_found` means the company could not be resolved to a live feed; pass the career-site URL directly and check the `didYouMean` suggestions.
- `job_not_found` on a single-job URL means the posting closed or expired.
- Zero rows usually means filters are too tight or `activeOnly` dropped expired postings; widen the filters.

## Related Actors

See `references/actor-index.md` for the companion `apify-successfactors-mcp` skill and the other ATS job APIs on this account (Workday, Greenhouse, Ashby, iCIMS, Oracle Taleo).
