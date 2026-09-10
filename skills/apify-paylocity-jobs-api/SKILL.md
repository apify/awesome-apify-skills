---
name: apify-paylocity-jobs-api
description: "Pull live jobs from any Paylocity recruiting board with the Apify Paylocity Jobs API Actor (johnvc/paylocity-jobs-api). Returns job rows as JSON: title, company, board GUID, department, full street address with city, state and ZIP, remote flag, published date, apply URL, the employer's own published pay as flat salaryMin, salaryMax, salaryCurrency and salaryPeriod columns, and the description as source HTML plus optional Markdown or plain text. Takes Paylocity board GUIDs, recruiting.paylocity.com URLs, or company names; filters by title, department, location, remote and employment type before billing; newerThan turns a daily schedule into a new-postings feed. Use when someone wants paylocity jobs data, a paylocity jobs api, to scrape a Paylocity career site or recruiting board, paylocity salary data, or postings from any employer that hires through Paylocity. No API key, billed per job delivered, and MCP-ready for Claude and other AI agents."
author: John Cole
author_url: https://github.com/johnisanerd
license: MIT
metadata:
  version: "1.0"
  keywords: ["paylocity", "paylocity jobs api", "paylocity jobs", "paylocity careers", "ats jobs", "job postings api", "jobs api", "job scraper", "mcp"]
---

# Paylocity Jobs, as Rows You Can Query

Board GUIDs, board URLs, or company names in, live Paylocity jobs out as structured rows, each with the employer's own published salary already flattened into numbers.

## When to use this skill

- Someone wants a Paylocity jobs API and found only the employer-side HR and payroll APIs they cannot sign up for.
- You are filling a job board, a market-intel dashboard, or a sourcing tool with roles from Paylocity-hosted recruiting boards.
- You need salary as numbers the employer actually published, not a string a human has to read or a model has to guess.
- You want a daily new-postings feed for a set of Paylocity boards.

Not for: finding which companies use Paylocity in the first place. Use the companion `apify-companies-using-paylocity` skill, built on the same Actor but shaped for discovery. See `references/actor-index.md`.

## What you get

One dataset row per job. `resultType` separates `job` rows from `error` rows.

Core fields:

- `id`, `url` (canonical, safe as a dedupe key), `applyUrl`, `title`
- `companyName`, `boardGuid`, `boardUrl`
- `department`, `location`, `address` (street, city, state, zip, county), `isRemote`, `indeedRemoteType`, `employmentType`, `isInternal`
- `datePublished`, `datePosted`
- flat `salaryMin`, `salaryMax`, `salaryCurrency`, `salaryPeriod`, `salaryUnit`, and `salaryRaw` (structured, verbatim)
- `snippet` (short listing preview), `descriptionHtml` (source, in the base row), `descriptionMarkdown`, `descriptionText` (opt-in add-ons)
- `source` (always `paylocity`), `sourceType` (`ats`), `scrapedAt`

The Actor ships dataset views for the console: `overview`, `salaries` (sortable pay columns), `companies`, and `newPostings`.

## Prerequisites

- Apify account (sign up at https://apify.com?fpr=9n7kx3&fp_sid=awesomeskills).
- Authentication via `apify login`, or an `APIFY_TOKEN` environment variable (Apify Console, Settings, Integrations).

## The Actor

- Store page: https://apify.com/johnvc/paylocity-jobs-api?fpr=9n7kx3&fp_sid=awesomeskills
- Actor ID: `johnvc/paylocity-jobs-api`
- Pricing: pay per event, no start fee, no API key. See the cost section below and `references/gotchas.md` for the live-price command.

## Run it with the Apify CLI

Every open job on one board, capped at 25, with Markdown descriptions:

```bash
apify actors call "johnvc/paylocity-jobs-api" -i '{"companies":["0062c37f-a34c-479c-978c-bd800d23f223"],"includeDescriptionMarkdown":true,"maxJobs":25}' \
  --json \
  --user-agent apify-awesome-skills/apify-paylocity-jobs-api \
  2>/dev/null
```

Remote roles across several boards, descriptions off for cheaper metadata rows:

```bash
apify actors call "johnvc/paylocity-jobs-api" -i '{"companies":["Indiana Health Centers","TAS Environmental Services"],"remoteOnly":true,"includeDescriptionMarkdown":false,"maxJobs":100}' \
  --json \
  --user-agent apify-awesome-skills/apify-paylocity-jobs-api \
  2>/dev/null
```

Only postings added in the last day, the shape to put on a daily schedule:

```bash
apify actors call "johnvc/paylocity-jobs-api" -i '{"companies":["0062c37f-a34c-479c-978c-bd800d23f223"],"newerThan":"25h","maxJobs":200}' \
  --json \
  --user-agent apify-awesome-skills/apify-paylocity-jobs-api \
  2>/dev/null
```

Confirm the live schema and prices before a large batch:

```bash
apify actors info "johnvc/paylocity-jobs-api" --json \
  --user-agent apify-awesome-skills/apify-paylocity-jobs-api \
  2>/dev/null
```

Read the rows back from a finished run:

```bash
apify datasets get-items <DATASET_ID> --format json \
  --user-agent apify-awesome-skills/apify-paylocity-jobs-api \
  2>/dev/null
```

Every call carries the three flags this repo expects: `--json` (or `--format json`), `--user-agent apify-awesome-skills/apify-paylocity-jobs-api`, and `2>/dev/null`.

## Run it from Claude or another AI agent (MCP)

The Actor is MCP-ready. Add the hosted server URL:

`https://mcp.apify.com/?tools=actors,docs,johnvc/paylocity-jobs-api`

Then ask, for example: "Pull the open jobs from this Paylocity board with published salary ranges, and rank them by salaryMax." MCP setup docs: https://docs.apify.com/platform/integrations/mcp

## Workflow

1. Start with one board and `maxJobs` around 25. Look at the row shape before you pay for a wide crawl.
2. Input a board GUID, a full `recruiting.paylocity.com/recruiting/jobs/All/{guid}/{slug}` URL, or a company name. Company names are resolved through discovery; an unresolved name returns a clear `error` row rather than guessing.
3. Filter at the source, not downstream. `titleKeywords`, `departments`, `locationKeywords`, `remoteOnly`, `employmentTypes`, and `newerThan` all drop jobs before they are billed.
4. Keep `includeDescriptionMarkdown` on for LLM pipelines; the base row already carries the source HTML description, so leave the add-ons off for lean rows.
5. For salary work, read the flat columns. `salaryMin` is null when the employer does not display pay; `salaryUnit` tells you hourly versus annual.
6. Check `resultType` before treating a row as a job. An `error` row carries a sanitized `errorMessage`.
7. Dedupe downstream on `url`. It is canonical and survives a company editing the title.

## Inputs

- `companies` (array): Paylocity board GUIDs, `recruiting.paylocity.com` URLs, or company names, mixed freely.
- `startUrls` (array): the same values in URL-list form; merged with `companies`.
- `titleKeywords`, `departments`, `locationKeywords` (arrays): keep-only filters, run before billing.
- `remoteOnly` (boolean, default false), `employmentTypes` (array)
- `newerThan` (string): `24h`, `7d`, `2w`, or an ISO date, compared against each posting's own timestamp.
- `cutoffField` (enum `posted`, `updated`): which timestamp `newerThan` uses.
- `includeDescriptionMarkdown`, `includeDescriptionText` (booleans, default false): per-row add-ons.
- `report` (enum `none`, `markdown`, `html`): a whole-run digest in the key-value store.
- `maxJobs` (integer, default 100): the primary spend cap. `maxJobsPerCompany`, `maxConcurrency` refine it.
- `proxyConfiguration` (object): off by default; direct connections work.

## Cost

Billing is pay per event with no start fee, so a run that returns nothing costs almost nothing. Confirm live prices with the info command above rather than trusting a number copied here.

The base `job-result` event fires once per delivered job row, source-HTML description and salary included. Add-on events fire only on rows that carry the extra: `job-description-markdown`, `job-description-text`, and a flat `run-report`. Jobs removed by your filters, and expired postings, are never charged.

Suggested confirmation thresholds: mention the estimate under about $5, warn the user over about $5, get explicit confirmation over about $20. Present cost as "around $X", never as a guarantee.

## Honest limits

- Salary appears only when the employer displays compensation on the posting. Nothing is inferred by a model.
- Public recruiting-board data only. No applicant data, no recruiter contacts, nothing behind a login.
- Company-name input depends on the board being discoverable in the public web archive. A brand-new board may need its GUID or URL until the archive catches up.
- One board is read live per run, so the rows reflect the board right now, not an index from last week.

## Troubleshooting

- An `error` row with a not-public message: the board GUID no longer resolves. Confirm the GUID from the live careers page.
- A company name that does not resolve: paste the board GUID or the full `recruiting.paylocity.com` URL instead.
- Zero rows and no error row: your filters removed everything. Relax `newerThan` or `titleKeywords` first.
- Duplicates across runs: dedupe on `url`, never on title.

See `references/gotchas.md` for cost guardrails and error recovery, and `references/actor-index.md` for the Actor routing table.

## Related Actors

- Greenhouse Job Board API: https://apify.com/johnvc/greenhouse-job-board-api?fpr=9n7kx3&fp_sid=awesomeskills
- Workday Careers API: https://apify.com/johnvc/workday-careers-api?fpr=9n7kx3&fp_sid=awesomeskills
- iCIMS Careers API: https://apify.com/johnvc/icims-careers-api?fpr=9n7kx3&fp_sid=awesomeskills
- Oracle and Taleo Jobs API: https://apify.com/johnvc/oracle-taleo-jobs-api?fpr=9n7kx3&fp_sid=awesomeskills
