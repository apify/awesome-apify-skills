---
name: apify-companies-using-paylocity
description: "Find companies that hire on Paylocity with the Apify Paylocity Jobs API Actor (johnvc/paylocity-jobs-api) in discovery mode. Enumerates Paylocity recruiting boards from the public web archive and returns one row per company: company name, board GUID, board URL, live open-jobs count, locations, and departments. Filter the set with a discovery query, live-verify each board, and cap the run. Use when someone wants a list of companies using Paylocity, companies hiring through Paylocity, Paylocity customer or prospect lists for sales and sourcing, or a directory of Paylocity career sites to feed lead generation or market research. No API key, billed per company row delivered, and MCP-ready for Claude and other AI agents."
author: John Cole
author_url: https://github.com/johnisanerd
license: MIT
metadata:
  version: "1.0"
  keywords: ["paylocity", "companies using paylocity", "companies hiring on paylocity", "paylocity customers", "lead generation", "sales prospecting", "ats discovery", "job board directory", "mcp"]
---

# Companies Using Paylocity, as a List You Can Work

Turn on discovery and get a directory of employers that hire on Paylocity: one row per company, each with a live open-jobs count, straight from the public web archive.

## When to use this skill

- You want a list of companies that use Paylocity, for a prospect list, a market map, or a partner search.
- Hiring is a buying signal, and you want to see who on Paylocity is actively posting roles in your niche.
- You are building a directory of Paylocity career sites to feed a sourcing tool or a CRM.
- You need the board GUID for a set of companies before pulling their jobs.

Not for: pulling the actual job postings from a board you already know. Use the companion `apify-paylocity-jobs-api` skill, built on the same Actor but shaped for extraction. See `references/actor-index.md`.

## What you get

One dataset row per company. `resultType` is `company` (with `error` rows for boards that could not be read).

Core fields:

- `companyName`, `boardGuid`, `boardUrl`
- `jobCount` (live open roles when `verifyLive` is on), `status` (`live`, `empty`, `dead`, or `unverified`)
- `locations` (the board's declared locations), `departments`
- `slugSource` (how the board was found), `discoveredAt`

Pair a discovered `boardGuid` with the `apify-paylocity-jobs-api` skill to pull that company's jobs.

## Prerequisites

- Apify account (sign up at https://apify.com?fpr=9n7kx3&fp_sid=awesomeskills).
- Authentication via `apify login`, or an `APIFY_TOKEN` environment variable (Apify Console, Settings, Integrations).

## The Actor

- Store page: https://apify.com/johnvc/paylocity-jobs-api?fpr=9n7kx3&fp_sid=awesomeskills
- Actor ID: `johnvc/paylocity-jobs-api`
- Pricing: pay per event, no start fee, no API key. Discovered live or empty boards are one cheap event each; unverified, dead, and error rows are free. See `references/gotchas.md`.

## Run it with the Apify CLI

Discover up to 25 Paylocity boards, live-verified, company rows only:

```bash
apify actors call "johnvc/paylocity-jobs-api" -i '{"discoverAll":true,"discoverOnly":true,"verifyLive":true,"maxCompanies":25}' \
  --json \
  --user-agent apify-awesome-skills/apify-companies-using-paylocity \
  2>/dev/null
```

Narrow the discovery to boards whose slug matches a term (for example a sector or a name fragment):

```bash
apify actors call "johnvc/paylocity-jobs-api" -i '{"discoverAll":true,"discoverOnly":true,"discoveryQuery":"health","verifyLive":true,"maxCompanies":50}' \
  --json \
  --user-agent apify-awesome-skills/apify-companies-using-paylocity \
  2>/dev/null
```

Widen the archive coverage by unioning more monthly snapshots:

```bash
apify actors call "johnvc/paylocity-jobs-api" -i '{"discoverAll":true,"discoverOnly":true,"crawlDepth":4,"maxCompanies":100}' \
  --json \
  --user-agent apify-awesome-skills/apify-companies-using-paylocity \
  2>/dev/null
```

Confirm the live schema and prices before a large sweep:

```bash
apify actors info "johnvc/paylocity-jobs-api" --json \
  --user-agent apify-awesome-skills/apify-companies-using-paylocity \
  2>/dev/null
```

Every call carries the three flags this repo expects: `--json` (or `--format json`), `--user-agent apify-awesome-skills/apify-companies-using-paylocity`, and `2>/dev/null`.

## Run it from Claude or another AI agent (MCP)

The Actor is MCP-ready. Add the hosted server URL:

`https://mcp.apify.com/?tools=actors,docs,johnvc/paylocity-jobs-api`

Then ask, for example: "Discover healthcare companies hiring on Paylocity, keep the boards with at least five open roles, and give me the company name plus board URL." MCP setup docs: https://docs.apify.com/platform/integrations/mcp

## Workflow

1. Set `discoverAll: true` and `discoverOnly: true` to get company rows and skip job scraping.
2. Keep `verifyLive: true` so each row carries a real open-jobs count and a `status`. Turn it off for a faster, cheaper first pass that skips the live probe.
3. Use `discoveryQuery` to narrow the set to a slug fragment, then raise `maxCompanies` once the shape looks right.
4. Raise `crawlDepth` to union more monthly web-archive snapshots when you want broader coverage; the trade-off is a slower sweep.
5. Filter downstream on `jobCount` and `status` to keep only boards that are actively posting.
6. Feed a chosen `boardGuid` into the `apify-paylocity-jobs-api` skill to pull that company's jobs.

## Inputs

- `discoverAll` (boolean): enumerate Paylocity boards from the public web archive.
- `discoverOnly` (boolean): return the company directory only, skip jobs.
- `discoveryQuery` (string): text match over discovered board slugs.
- `crawlDepth` (integer, default 2): how many monthly web-archive snapshots to union.
- `verifyLive` (boolean, default true): live-probe each board for a current open-jobs count and status.
- `includeInactive` (boolean, default false): also return dead boards.
- `maxCompanies` (integer, default 25): the primary spend cap for a discovery run.
- `maxConcurrency` (integer): parallel verification requests.

## Cost

Billing is pay per event with no start fee. Each live or empty company row is one cheap `company-result` event; unverified, dead, and error rows are delivered free. `maxCompanies` is the spend cap. Confirm live prices with the info command above rather than trusting a number copied here.

Suggested confirmation thresholds: mention the estimate under about $5, warn the user over about $5, get explicit confirmation over about $20. Present cost as "around $X", never as a guarantee.

## Honest limits

- Discovery reads the public web archive, so a brand-new board can lag by a few weeks until the archive catches up. Known GUIDs and URLs always work immediately in the extraction skill.
- The archive is broad but not a guaranteed census; `crawlDepth` widens coverage at the cost of a slower run.
- `jobCount` reflects the live board at run time when `verifyLive` is on; with it off, the row is the archive record without a fresh count.
- Public recruiting-board data only. No applicant data and nothing behind a login.

## Troubleshooting

- Few rows on a narrow `discoveryQuery`: broaden the term or raise `crawlDepth`.
- Many `unverified` rows: turn `verifyLive` on to get counts and a real `status`.
- A discovered board returns no jobs later: its `status` was `empty`; the company has no open roles right now.
- Duplicates across sweeps: dedupe on `boardGuid`.

See `references/gotchas.md` for cost guardrails and error recovery, and `references/actor-index.md` for the Actor routing table.

## Related Actors

- Greenhouse Job Board API: https://apify.com/johnvc/greenhouse-job-board-api?fpr=9n7kx3&fp_sid=awesomeskills
- Workday Career Sites API: https://apify.com/johnvc/workday-career-sites-api?fpr=9n7kx3&fp_sid=awesomeskills
- iCIMS Careers API: https://apify.com/johnvc/icims-careers-api?fpr=9n7kx3&fp_sid=awesomeskills
- Ashby Job Board API: https://apify.com/johnvc/ashby-job-board-scraper?fpr=9n7kx3&fp_sid=awesomeskills
