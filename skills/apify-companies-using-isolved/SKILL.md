---
name: apify-companies-using-isolved
description: "Find companies that use isolved, and check whether a specific employer has an isolved job board, with the isolved Jobs API Actor (johnvc/isolved-jobs-api). Discovery mode returns one row per employer hiring through isolved: tenant slug, careers URL, and a live open-jobs count, drawn from a directory rebuilt from isolved's own public sitemap index of every tenant and re-probed at run time so dead tenants are skipped and never billed. Feed it your own prospect list instead and tenant slugs or URLs are verified live, and a miss returns a clear not-found row. Use when someone asks for companies that use isolved, a list of employers using isolvedhire, isolved customers, who hires through isolved, to check if a company uses isolved or ApplicantPro, or to build ATS-based sales intelligence and recruiting target lists. isolved skews to United States small and mid-market employers, so the directory doubles as an SMB lead list. Billed per verified tenant row, no start fee, MCP-ready for AI agents."
author: John Cole
author_url: https://github.com/johnisanerd
license: MIT
metadata:
  version: "1.0"
---

# Companies That Use isolved, as a Live Directory

A query or a prospect list in, live-verified employers on isolved out, each with its careers URL and a current open-jobs count.

## When to use this skill

- Someone asks which companies use isolved, for a lead list, market map, or recruiting target list.
- You have a prospect list and need to know who hires through isolved before pointing a jobs pipeline at them.
- You want hiring as a buying signal: isolved skews to US small and mid-market employers, and open-roles counts show where headcount is going.
- You need to verify one employer: does it have an isolved career site at all, and what is the URL?

Not for: pulling the job postings themselves. Use the companion `apify-isolved-jobs-api` skill, built on the same Actor but shaped for job rows. See `references/actor-index.md`.

## What you get

One dataset row per tenant. `resultType` separates `tenant` rows from `error` rows.

- `organization`: the employer name, from the posting or derived from the tenant.
- `tenant`: the tenant slug, the subdomain before `.isolvedhire.com`.
- `tenantUrl`: the public careers site.
- `jobCount`: the current live open-jobs count from the tenant's sitemap.
- `live`: whether the tenant was verified against its live sitemap this run.
- `region`, `verifiedAt`, `scrapedAt`.

## Prerequisites

- Apify account (sign up at https://apify.com?fpr=9n7kx3&fp_sid=awesomeskills).
- Authentication via `apify login`, or an `APIFY_TOKEN` environment variable (Apify Console, Settings, Integrations).

## The Actor

- Store page: https://apify.com/johnvc/isolved-jobs-api?fpr=9n7kx3&fp_sid=awesomeskills
- Actor ID: `johnvc/isolved-jobs-api`
- Pricing: pay per event, no start fee. See the cost section below and `references/gotchas.md` for the live-price command.

## Run it with the Apify CLI

The largest tenants first, verified live, capped at 50:

```bash
apify actors call "johnvc/isolved-jobs-api" -i '{"outputMode":"tenantsOnly","maxTenants":50}' \
  --json \
  --user-agent apify-awesome-skills/apify-companies-using-isolved \
  2>/dev/null
```

Scope the directory to a query:

```bash
apify actors call "johnvc/isolved-jobs-api" -i '{"outputMode":"tenantsOnly","discoveryQuery":"oil","maxTenants":25}' \
  --json \
  --user-agent apify-awesome-skills/apify-companies-using-isolved \
  2>/dev/null
```

Verify your own prospect list of tenant slugs:

```bash
apify actors call "johnvc/isolved-jobs-api" -i '{"outputMode":"tenantsOnly","tenants":["davidsonoil","goflyingstar","isolved"]}' \
  --json \
  --user-agent apify-awesome-skills/apify-companies-using-isolved \
  2>/dev/null
```

Confirm the live schema and prices before a large sweep:

```bash
apify actors info "johnvc/isolved-jobs-api" --json \
  --user-agent apify-awesome-skills/apify-companies-using-isolved \
  2>/dev/null
```

Read the rows back from a finished run:

```bash
apify datasets get-items <DATASET_ID> --format json \
  --user-agent apify-awesome-skills/apify-companies-using-isolved \
  2>/dev/null
```

Every call carries the three flags this repo expects: `--json` (or `--format json`), `--user-agent apify-awesome-skills/apify-companies-using-isolved`, and `2>/dev/null`.

## Run it from Claude or another AI agent (MCP)

The Actor is MCP-ready. Add the hosted server URL:

`https://mcp.apify.com/?tools=actors,docs,johnvc/isolved-jobs-api`

Then ask, for example: "List employers hiring through isolved with the most open roles, and give me their careers URLs." MCP setup docs: https://docs.apify.com/platform/integrations/mcp

## Workflow

1. For a market map, run `tenantsOnly` with no `tenants` and a `maxTenants` cap; the directory returns largest tenants first.
2. For a prospect check, pass your own `tenants` list. Each is verified against its live sitemap; a miss returns a `board_not_found` error row rather than a false positive.
3. `discoveryQuery` scopes the sweep by a substring of the tenant slug or name.
4. Keep `verifyTenants` on (default) for live open-jobs counts; turn it off to list the directory snapshot without network checks, faster and cheaper.
5. Read `jobCount` as a hiring-volume signal; `live: true` confirms the tenant answered this run.
6. Check `resultType` before treating a row as a tenant. An `error` row carries `errorCode` and `errorMessage`.
7. To pull the actual postings for a tenant you found, hand its slug to `apify-isolved-jobs-api`.

## Inputs

- `outputMode` (enum): set to `tenantsOnly` for this skill.
- `tenants` (array): your own tenant slugs or URLs to verify; empty sweeps the bundled directory.
- `discoveryQuery` (string): substring match over tenant slug and name.
- `verifyTenants` (boolean, default true): live-probe each tenant for a current open-jobs count.
- `maxTenants` (integer, default 25): the primary spend cap for discovery.
- `proxyConfiguration` (object): off by default; direct connections work.

## Cost

Billing is pay per event with no start fee. The `tenant-row` event fires once per live-verified tenant returned; dead tenants are skipped and never billed. Confirm live prices with the info command above rather than trusting a number copied here.

Suggested confirmation thresholds: mention the estimate under about $5, warn the user over about $5, get explicit confirmation over about $20. Present cost as "around $X", never as a guarantee.

## Honest limits

- The directory is rebuilt from isolved's public sitemap index, so it reflects tenants that publish a public job sitemap. An employer on isolved with no public postings may not appear.
- `jobCount` is the live count at run time and moves as employers post and close roles.
- Public career-site data only. No applicant data, no recruiter contacts, nothing behind a login.
- US-focused. isolved career sites are almost entirely United States employers.

## Troubleshooting

- `board_not_found` on a prospect check: that tenant has no public job sitemap; confirm the slug or paste the tenant URL.
- `http_error`: transient upstream answer; retry once before assuming anything.
- Zero rows: the `discoveryQuery` matched nothing, or `maxTenants` is too low. Widen the query.
- Duplicates: dedupe on `tenant`, the stable slug.

See `references/gotchas.md` for cost guardrails and error recovery, and `references/actor-index.md` for the Actor routing table.

## Related Actors

- Greenhouse Job Board API: https://apify.com/johnvc/greenhouse-job-board-api?fpr=9n7kx3&fp_sid=awesomeskills
- Ashby Job Board API: https://apify.com/johnvc/ashby-job-board-scraper?fpr=9n7kx3&fp_sid=awesomeskills
- iCIMS Careers API: https://apify.com/johnvc/icims-careers-api?fpr=9n7kx3&fp_sid=awesomeskills
- Workday Careers API: https://apify.com/johnvc/workday-careers-api?fpr=9n7kx3&fp_sid=awesomeskills
