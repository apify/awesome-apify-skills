---
name: apify-ashby-jobs-scraper
description: "Scrape Ashby jobs from any jobs.ashbyhq.com board with the Apify Ashby Job Board API Actor (johnvc/ashby-job-board-scraper). Returns live job rows as JSON: title, company, department, team, employment type, workplace type, remote flag, locations with derived countries, publish date, apply URL, the employer's own published pay as flat salaryMin, salaryMax, salaryCurrency and salaryPeriod columns plus an offersEquity flag, and the description as Markdown by default. Takes company names, board slugs, or board URLs; filters by title, department, location, employment type, and remote before billing; publishedAfter turns a daily schedule into a new-postings feed. Use when someone wants ashby jobs data, an ashby jobs api, to scrape ashby job postings for a job board or aggregator, ashby salary data, or listings from OpenAI, Ramp, Notion, Linear, Cerebras and other Ashby-hosted careers pages. Billed per job delivered with no start fee, and MCP-ready for Claude and other AI agents."
author: John Cole
author_url: https://github.com/johnisanerd
license: MIT
metadata:
  version: "1.0"
  keywords: "ashby, jobs, job-board, salary, remote-work"
---

# Ashby Jobs, as Rows You Can Query

Company names or board slugs in, live Ashby jobs out as structured rows, each with the employer's own published salary already flattened into numbers.

## When to use this skill

- Someone wants Ashby jobs and does not want to read jobs.ashbyhq.com boards by hand.
- You are filling a job board, a market-intel dashboard, or a sourcing tool with roles from Ashby-hosted careers pages.
- You need salary as numbers the employer actually published, not a string a human has to read or a model has to guess.
- You went looking for an Ashby jobs API and found only the employer-side APIs you cannot sign up for.

Not for: finding which companies run Ashby in the first place, or checking a prospect list. If installed, use the companion `apify-companies-using-ashby` skill, built on the same Actor but shaped for discovery. See `references/actor-index.md`.

## What you get

One dataset row per job. `resultType` separates `job` rows from `error` rows.

Core fields:

- `id`, `url` (canonical, safe as a dedupe key), `applyUrl`, `title`
- `companyName`, `boardToken`, `boardUrl`
- `department`, `team`, `departments`, `employmentType`, `workplaceType`, `isRemote`
- `location`, `secondaryLocations`, `locationsDerived`, `countriesDerived`, `address`
- `datePublished`, `isListed`
- `compensationSummary` (the employer's own pay line), `salaryRaw` (structured, verbatim), `salaryDerived`, and flat `salaryMin`, `salaryMax`, `salaryCurrency`, `salaryPeriod`, `salaryTiers`, `offersEquity`
- `descriptionMarkdown` (default add-on), `descriptionHtml`, `descriptionText` (opt-in add-ons)
- `source` (always `ashby`), `sourceType`, `scrapedAt`

With `includeCompanyData` on, each row also carries `companyWebsite`, `companyLogo`, `remoteEligibility` (the countries remote applicants may live in, as the employer declared them), `applicationDeadline`, and `directApply`.

The Actor ships dataset views for the console: `overview`, `salaries` (sortable pay columns), `companies`, and `newPostings`.

## Prerequisites

- Apify account (sign up at https://apify.com?fpr=9n7kx3&fp_sid=awesomeskills).
- Authentication via `apify login`, or an `APIFY_TOKEN` environment variable (Apify Console, Settings, Integrations).

## The Actor

- Store page: https://apify.com/johnvc/ashby-job-board-scraper?fpr=9n7kx3&fp_sid=awesomeskills
- Actor ID: `johnvc/ashby-job-board-scraper`
- Pricing: pay per event, no start fee. See the cost section below and `references/gotchas.md` for the live-price command.

## Run it with the Apify CLI

Engineering roles from two boards, capped at 25:

```bash
apify actors call "johnvc/ashby-job-board-scraper" -i '{"companies":["cerebras","ramp"],"titleKeywords":["engineer"],"maxJobs":25}' \
  --json \
  --user-agent apify-awesome-skills/apify-ashby-jobs-scraper \
  2>/dev/null
```

Remote full-time roles with pay, descriptions off for cheaper metadata rows:

```bash
apify actors call "johnvc/ashby-job-board-scraper" -i '{"companies":["openai","ramp","deel"],"remoteOnly":true,"employmentTypes":["FullTime"],"includeDescriptionMarkdown":false,"maxJobs":100}' \
  --json \
  --user-agent apify-awesome-skills/apify-ashby-jobs-scraper \
  2>/dev/null
```

Only postings added in the last day, the shape to put on a daily schedule:

```bash
apify actors call "johnvc/ashby-job-board-scraper" -i '{"companies":["openai","notion","linear"],"publishedAfter":"25h","maxJobs":200}' \
  --json \
  --user-agent apify-awesome-skills/apify-ashby-jobs-scraper \
  2>/dev/null
```

Confirm the live schema and prices before a large batch:

```bash
apify actors info "johnvc/ashby-job-board-scraper" --json \
  --user-agent apify-awesome-skills/apify-ashby-jobs-scraper \
  2>/dev/null
```

Read the rows back from a finished run:

```bash
apify datasets get-items <DATASET_ID> --format json \
  --user-agent apify-awesome-skills/apify-ashby-jobs-scraper \
  2>/dev/null
```

Every call carries the three flags this repo expects: `--json` (or `--format json`), `--user-agent apify-awesome-skills/apify-ashby-jobs-scraper`, and `2>/dev/null`.

## Run it from Claude or another AI agent (MCP)

The Actor is MCP-ready. Add the hosted server URL:

`https://mcp.apify.com/?tools=actors,docs,johnvc/ashby-job-board-scraper`

Then ask, for example: "Pull the remote engineering jobs from OpenAI and Ramp with published salary ranges, and rank them by salaryMax." MCP setup docs: https://docs.apify.com/platform/integrations/mcp

## Workflow

1. Start with one or two boards and `maxJobs` around 25. Look at the row shape before you pay for a wide crawl.
2. Company names work as input. "Black Semiconductor" finds the blacksemiconductor board; the Actor tries slug spellings automatically and returns a `board_not_found` error row with `didYouMean` suggestions on a miss.
3. Filter at the source, not downstream. `titleKeywords`, `departments`, `locationKeywords`, `employmentTypes`, `remoteOnly`, and `publishedAfter` all drop jobs before they are billed. For fully remote requests, also check `workplaceType`: `remoteOnly` can return Hybrid jobs.
4. Keep `includeDescriptionMarkdown` on for LLM pipelines; turn it off for metadata-only rows at roughly half the per-row cost.
5. For salary work, read the flat columns and `salaryDerived.source`. Published ranges can come from native compensation data or description parsing, even when `shouldDisplayCompensation` is false. Do not infer missing pay.
6. Check `resultType` before treating a row as a job. An `error` row carries `errorCode` and a human-readable `errorMessage`.
7. Dedupe downstream on `url`. It is canonical and survives a company editing the title.

## Inputs

- `companies` (array): company names, board slugs, or jobs.ashbyhq.com URLs (board, embed, or single job), mixed freely. Empty sweeps the bundled directory.
- `startUrls` (array): the same values in URL-list form; merged with `companies`.
- `outputMode` (enum `jobs`, `urlsOnly`, `companiesOnly`, default `jobs`)
- `titleKeywords`, `departments`, `locationKeywords` (arrays): keep-only filters, run before billing.
- `employmentTypes` (array of `FullTime`, `PartTime`, `Intern`, `Contract`, `Temporary`)
- `remoteOnly` (boolean, default false)
- `publishedAfter` (string): `24h`, `7d`, `2w`, or an ISO date.
- `includeDescriptionMarkdown` (boolean, default true), `includeDescriptionHtml`, `includeDescriptionText` (default false)
- `includeCompanyData` (boolean, default false): job-page enrichment, one extra request per job.
- `report` (enum `none`, `markdown`, `html`): a whole-run digest in the key-value store.
- `maxJobs` (integer, default 100): the primary spend cap. `maxJobsPerCompany`, `maxCompanies`, `maxConcurrency` refine it.
- `proxyConfiguration` (object): off by default; direct connections work.

## Cost

Billing is pay per event with no start fee, so a run that returns nothing costs almost nothing. Confirm live prices with the info command above rather than trusting a number copied here.

The base `job-result` event fires once per delivered job row, salary data included. Add-on events fire only on rows that carry the extra: `job-description-markdown` (on by default), `job-description-html`, `job-description-text`, `job-company-data`, and a flat `run-report`. Company discovery rows and URL index rows have their own cheaper events.

Jobs removed by your filters are never charged. As a shape, 100 jobs with Markdown descriptions is small change.

Suggested confirmation thresholds: mention the estimate under about $5, warn the user over about $5, get explicit confirmation over about $20. Present cost as "around $X", never as a guarantee.

## Honest limits

- **No update timestamps exist.** Ashby publishes only `publishedAt`, so `publishedAfter` gives you a new-postings feed; there is no changed-jobs feed and no field pretends otherwise.
- Salary appears only when the employer displays compensation on the posting. Nothing is inferred by a model; `salaryDerived.source` tells you whether a value came from the structured data or a deterministic text parse.
- Public board data only. No applicant data, no recruiter contacts, nothing behind a login.
- One board is one request, so very large boards return in a single response; a board too large to process returns a clear in-band error rather than a partial dataset.

## Troubleshooting

- `board_not_found`: the slug does not exist on the public API. Check `didYouMean` on the error row, or paste the board URL instead of a name.
- `job_not_found`: a single-job URL points at a posting that closed. Fetch the whole board instead.
- `http_error`: the source answered abnormally; the row says which HTTP status. Retry once before assuming anything.
- Zero rows and no error row: your filters removed everything. Relax `publishedAfter` or `titleKeywords` first.
- Duplicates across runs: dedupe on `url`, never on title.

See `references/gotchas.md` for cost guardrails and error recovery, and `references/actor-index.md` for the Actor routing table.

## Related Actors

- Greenhouse Job Board API: https://apify.com/johnvc/greenhouse-job-board-api?fpr=9n7kx3&fp_sid=awesomeskills
- Workday Careers API: https://apify.com/johnvc/workday-careers-api?fpr=9n7kx3&fp_sid=awesomeskills
- Wellfound Jobs API: https://apify.com/johnvc/wellfound-jobs-api?fpr=9n7kx3&fp_sid=awesomeskills
- LinkedIn Jobs API: https://apify.com/johnvc/linkedin-jobs-api?fpr=9n7kx3&fp_sid=awesomeskills
