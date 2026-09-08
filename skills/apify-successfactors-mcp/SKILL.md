---
name: apify-successfactors-mcp
description: "Use live SAP SuccessFactors job data from Claude and other MCP clients with the Apify SAP SuccessFactors Jobs API Actor (johnvc/sap-successfactors-jobs-api), exposed through the hosted Apify MCP server. Adds a tool that pulls postings from any public SuccessFactors career site and returns structured JSON: title, company, requisition ID, job function, location, expiration date, and the description as Markdown, with an optional detail add-on for posted date, department, facility, shift type, and travel. Use when someone wants a successfactors mcp server, to connect SuccessFactors to Claude AI, to give an AI agent live SAP SuccessFactors jobs, or to query enterprise job postings from an MCP client without writing glue code. No key needed for the public jobs layer, and billed per job delivered."
author: John Cole
author_url: https://github.com/johnisanerd
license: MIT
metadata:
  version: "1.0"
---

# SAP SuccessFactors Jobs, Inside Claude

Give Claude or any MCP client a tool that reads live SAP SuccessFactors postings on demand and hands back structured rows to reason over, with no glue code.

## When to use this skill

- You want a SuccessFactors MCP server that reads career sites, not one that manages an employer tenant.
- You are connecting SuccessFactors to Claude AI and want live postings an agent can pull mid-conversation.
- Your agent needs enterprise job data (roles, locations, functions) as structured JSON, on request.
- You do not want to build and host your own MCP wrapper around a SuccessFactors scraper.

Not for: bulk one-off exports from a script or a scheduled pipeline. Use the companion `apify-successfactors-jobs-api` skill, built on the same Actor. See `references/actor-index.md`.

## What you get

The Actor returns one row per job (`resultType` separates `job` from `error` rows):

- `requisitionId`, `url`, `applyUrl`, `title`
- `companyName`, `employer`, `tenant`, `careerSiteUrl`
- `jobFunction`, `location`, `countryCode`, `isRemote`, `validThrough`
- `descriptionMarkdown` by default, with `descriptionHtml` and `descriptionText` opt-in
- with `includeDetailFields`: `datePosted`, `department`, `facility`, `shiftType`, `travel`, `customFields`
- `source` (always `successfactors`), `sourceType`, `flavor`, `scrapedAt`

## Prerequisites

- Apify account (sign up at https://apify.com?fpr=9n7kx3&fp_sid=awesomeskills).
- An `APIFY_TOKEN` for the hosted MCP server, or a Claude client that signs in to Apify.

## The Actor

- Store page: https://apify.com/johnvc/sap-successfactors-jobs-api?fpr=9n7kx3&fp_sid=awesomeskills
- Actor ID: `johnvc/sap-successfactors-jobs-api`
- Pricing: pay per event, no start fee. See the cost section and `references/gotchas.md`.

## Run it from Claude (MCP)

Add the hosted Apify MCP server with this Actor in the tools list:

```
https://mcp.apify.com/?tools=actors,docs,johnvc/sap-successfactors-jobs-api
```

In [Claude Code](https://claude.ai/referral/uIlpa7nPLg) (free trial) or [Claude Cowork](https://claude.ai/referral/uIlpa7nPLg) (free trial), add it as an MCP server, then ask Claude to call the SAP SuccessFactors Jobs API tool with an input like:

```json
{"companies":["jobs.sap.com"],"maxJobs":25,"includeDescriptionMarkdown":true}
```

Claude discovers the tool, runs it, and reads the dataset back with no glue code. Docs: https://docs.apify.com/platform/integrations/mcp

## CLI fallback

If you would rather drive it from a shell:

```bash
apify actors call "johnvc/sap-successfactors-jobs-api" \
  -i '{"companies":["jobs.sap.com"],"maxJobs":25}' \
  --json \
  --user-agent apify-awesome-skills/apify-successfactors-mcp \
  2>/dev/null
```

## Workflow

1. Add the hosted MCP server URL above to your MCP client.
2. Ask the agent to call the Actor tool with the career sites you care about (a host, URL, or company name).
3. Keep the first call small (`maxJobs` of 10 to 25) so the agent gets a fast, cheap sample.
4. Have the agent filter with `titleKeywords`, `jobFunctions`, or `locationKeywords` before scaling up; filters run before billing.
5. Read the returned rows and reason over them: rank roles, extract skills, compare locations, or write them to storage.

## Inputs

- `companies`, `startUrls`: career-site hosts, URLs, single-job URLs, or company names.
- `outputMode`: `jobs` (default), `urlsOnly`, or `tenantsOnly`.
- `titleKeywords`, `jobFunctions`, `locationKeywords`, `activeOnly`, `remoteOnly`: pre-billing filters.
- `includeDescriptionMarkdown` / `Html` / `Text`, `includeDetailFields`, `report`: output add-ons.
- `maxJobs`, `maxJobsPerTenant`, `maxTenants`, `maxConcurrency`: scope and cost caps.

## Cost guardrails

- Per delivered row, no start fee; filters run before billing so filtered rows cost nothing.
- Description formats, detail fields, and the run report are separate add-ons billed only on rows that carry them.
- For agent use, keep `maxJobs` low per call and let the agent widen only when the sample looks right.

## Honest limits

- The MCP path runs the same Actor as a script would; it does not add fields, it changes how you call it.
- The public feed has no posted date; `datePosted` needs the detail add-on.
- Salary is a best-effort parse of the posting text and is often null.

## Troubleshooting

- If the tool does not appear, confirm the Actor slug in the `?tools=` URL and that the client is signed in to Apify.
- An `error` row with `tenant_not_found` means the company did not resolve to a live feed; pass the career-site URL.
- Large runs are slower with `includeDetailFields` on (one extra request per job); lower `maxJobs` for interactive use.

## Related Actors

See `references/actor-index.md` for the companion `apify-successfactors-jobs-api` skill and the other ATS job APIs on this account.
