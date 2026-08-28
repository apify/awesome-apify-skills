---
name: apify-companies-using-ashby
description: "Find companies that use Ashby, and check whether any specific company has an Ashby job board, with the Apify Ashby Job Board API Actor (johnvc/ashby-job-board-scraper). Discovery mode returns one row per company hiring through Ashby: company name, board slug, board URL, and a live open-jobs count, drawn from a bundled directory of 2,700 plus verified boards and re-probed at run time so dead boards are skipped and never billed. Feed it your own prospect list instead and plain company names are matched to board slugs automatically; a miss returns a clear not-found row with did-you-mean suggestions. Use when someone asks for companies that use ashby, a list of companies using ashbyhq, ashby customers, who hires through Ashby, to check if a company uses Ashby, or to build ATS-based sales intelligence and recruiting target lists. Hiring through Ashby skews to fast-growing startups, so the directory doubles as a growth-company lead list. Billed per verified company row, no start fee, MCP-ready for AI agents."
author: John Cole
author_url: https://github.com/johnisanerd
license: MIT
metadata:
  version: "1.0"
---

# Companies That Use Ashby, as a Live Directory

A query or a prospect list in, live-verified companies on Ashby out, each with its board URL and a current open-jobs count.

## When to use this skill

- Someone asks which companies use Ashby, for a lead list, market map, or recruiting target list.
- You have a prospect list and need to know who hires through Ashby before pointing a jobs pipeline at them.
- You want hiring as a buying signal: companies on Ashby skew to funded, fast-growing startups, and open-roles counts show where headcount is going.
- You need to verify one company: does it have an Ashby job board at all, and what is the board URL?

Not for: pulling the job postings themselves. Use the companion `apify-ashby-jobs-scraper` skill, built on the same Actor but shaped for job rows with salary data. See `references/actor-index.md`.

## What you get

One dataset row per company. `resultType` separates `company` rows from `error` rows.

- `companyName`: the company's name from the bundled directory.
- `boardToken`: the board slug, the last path segment of the board URL.
- `boardUrl`: the public jobs.ashbyhq.com board.
- `jobCount`: live open roles at verification time.
- `live`: whether the board answered the probe (null when verification is off).
- `verifiedAt`, `scrapedAt` (ISO timestamps)

A not-found check returns an `error` row with `errorCode: board_not_found`, a plain `errorMessage`, and `didYouMean`: the closest known board slugs.

The Actor ships a `companies` dataset view that puts name, slug, open jobs, live flag, and the board link on screen in that order.

## Prerequisites

- Apify account (sign up at https://apify.com?fpr=9n7kx3&fp_sid=awesomeskills).
- Authentication via `apify login`, or an `APIFY_TOKEN` environment variable (Apify Console, Settings, Integrations).

## The Actor

- Store page: https://apify.com/johnvc/ashby-job-board-scraper?fpr=9n7kx3&fp_sid=awesomeskills
- Actor ID: `johnvc/ashby-job-board-scraper`
- Pricing: pay per event, no start fee; a discovered company row is its own cheap event. See `references/gotchas.md` for the live-price command.

## Run it with the Apify CLI

The hundred biggest Ashby boards, live-verified:

```bash
apify actors call "johnvc/ashby-job-board-scraper" -i '{"outputMode":"companiesOnly","maxCompanies":100}' \
  --json \
  --user-agent apify-awesome-skills/apify-companies-using-ashby \
  2>/dev/null
```

Narrow the directory by name, for a themed list:

```bash
apify actors call "johnvc/ashby-job-board-scraper" -i '{"outputMode":"companiesOnly","discoveryQuery":"ai","maxCompanies":50}' \
  --json \
  --user-agent apify-awesome-skills/apify-companies-using-ashby \
  2>/dev/null
```

Check a prospect list by plain company names, misses included:

```bash
apify actors call "johnvc/ashby-job-board-scraper" -i '{"companies":["Black Semiconductor","Morse Micro","Ramp","Notarealcompany Xyz"],"outputMode":"companiesOnly"}' \
  --json \
  --user-agent apify-awesome-skills/apify-companies-using-ashby \
  2>/dev/null
```

Confirm the live schema and prices before a large batch:

```bash
apify actors info "johnvc/ashby-job-board-scraper" --json \
  --user-agent apify-awesome-skills/apify-companies-using-ashby \
  2>/dev/null
```

Read the rows back from a finished run:

```bash
apify datasets get-items <DATASET_ID> --format json \
  --user-agent apify-awesome-skills/apify-companies-using-ashby \
  2>/dev/null
```

Every call carries the three flags this repo expects: `--json` (or `--format json`), `--user-agent apify-awesome-skills/apify-companies-using-ashby`, and `2>/dev/null`.

## Run it from Claude or another AI agent (MCP)

The Actor is MCP-ready. Add the hosted server URL:

`https://mcp.apify.com/?tools=actors,docs,johnvc/ashby-job-board-scraper`

Then ask, for example: "Which of these twenty companies hire through Ashby, and how many roles does each have open right now?" MCP setup docs: https://docs.apify.com/platform/integrations/mcp

## Workflow

1. Decide which question you are asking. A market map starts from the directory (`discoveryQuery` plus `maxCompanies`); a prospect check starts from your own `companies` list.
2. For prospect checks, pass names the way you know them. The Actor tries slug spellings automatically ("Black Semiconductor" finds blacksemiconductor) and returns `didYouMean` suggestions on a miss.
3. Keep `verifyCompanies` on. Live probing is what makes `jobCount` current and keeps dead boards out of your list, and skipped dead boards are never billed.
4. Sort by `jobCount` downstream. Open-roles count is the growth signal; a two-job board and a two-hundred-job board are different prospects.
5. To go from companies to their postings, hand the `boardToken` values to the companion `apify-ashby-jobs-scraper` skill.
6. Re-run monthly rather than caching. Boards appear and die; the whole point of the live probe is freshness.

## Inputs

- `outputMode`: set `companiesOnly` for this workflow.
- `companies` (array): your own prospect list; names, slugs, or board URLs.
- `discoveryQuery` (string): case-insensitive text match over directory names and slugs.
- `verifyCompanies` (boolean, default true): live-probe each candidate; dead boards are skipped and never billed.
- `maxCompanies` (integer, default 25): the primary cap for directory sweeps.
- `maxConcurrency` (integer, default 5): parallel probes.

## Cost

A verified company row is a single cheap `company-row` event, and there is no start fee, so checking a fifty-name prospect list costs well under a cent. Dead directory entries and misses on your own list are never billed as companies. Confirm live prices with the info command above.

Suggested confirmation thresholds: mention the estimate under about $5, warn over $5, confirm over $20. At this event's price you will rarely leave the first bracket.

## Honest limits

- The bundled directory covers 2,700 plus verified boards found in public web archives. It is refreshed with releases and re-probed live at run time, but a company that launched its board yesterday may not be in the directory yet; a direct check by name still finds it.
- Company names come from the directory. A board not in the directory returns its slug as the name.
- One company can run multiple boards under different slugs; each slug is its own row.
- Public board presence only. This says a company hires through Ashby; it says nothing about contract value, seats, or tenure.

## Troubleshooting

- `board_not_found` on a name you know is right: read `didYouMean`, or find the company's careers page and paste the jobs.ashbyhq.com URL it links to.
- Empty sweep: your `discoveryQuery` matched nothing; loosen it or drop it to take the largest boards first.
- `jobCount` null with `live` null: verification was turned off; the row is a directory snapshot, not a live read.
- Slow runs on big sweeps: raise `maxConcurrency` toward 10, or lower `maxCompanies`.

See `references/gotchas.md` for cost guardrails and error recovery, and `references/actor-index.md` for the Actor routing table.

## Related Actors

- Greenhouse Job Board API (companies using Greenhouse): https://apify.com/johnvc/greenhouse-job-board-api?fpr=9n7kx3&fp_sid=awesomeskills
- Crunchbase Company API: https://apify.com/johnvc/crunchbase-company-api?fpr=9n7kx3&fp_sid=awesomeskills
- LinkedIn Company API: https://apify.com/johnvc/linkedin-company-api?fpr=9n7kx3&fp_sid=awesomeskills
- PitchBook Company API: https://apify.com/johnvc/pitchbook-company-api?fpr=9n7kx3&fp_sid=awesomeskills
