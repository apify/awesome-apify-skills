# Gotchas — apify-reddit-scraper

Cost guardrails, error recovery and the quirks that cost real runs. Read on demand.

## Cost guardrails

`harshmaur/reddit-scraper` is `PAY_PER_EVENT`:

| Event | Price |
|---|---|
| Actor start | $0.02 per run |
| Result stored (post, comment, community or user row) | $0.002 each ($2.00 per 1,000); $1.50 per 1,000 on plans with Store discounts |
| AI analysis (`aiAnalysis: true`, paid plans only) | +$0.50 per 1,000 analysed rows |
| Custom AI label (`customLabels`, paid plans only) | +$0.10 per 1,000 evaluations per label |

Estimate before running:

```
items ≈ posts + (crawlCommentsPerPost ? posts × maxCommentsPerPost : 0) + (searchComments ? maxCommentsCount : 0)
cost  ≈ $0.02 + items × $0.002
```

| Estimate | Action |
|---|---|
| < $5 (≈ 2,500 items) | Run. |
| $5 – $20 | Say the estimate, run unless the user objects. |
| > $20 | Ask for explicit confirmation and offer a smaller first pass (`maxPostsCount: 100`, comments off). |

Free Apify plans: results beyond the plan's monthly credit stop the crawl (the Actor halts when the platform's charge limit is hit), and only the first 40 `searchTerms` per run are searched.

## The traps

1. **Post permalinks in `subredditUrls` are dropped silently.** The run SUCCEEDS with 0 items and no error. Permalinks belong in `startUrls`.
2. **Reddit search is literal.** Long descriptive phrases return nothing. Convert topics into short keywords and run several `searchTerms`.
3. **~1,000-post ceiling per listing.** Reddit stops paginating any search or subreddit listing around 1,000 posts regardless of `maxPostsCount`. Use `postedAfter`/`postedBefore` windows to walk further back; posts older than the ceiling in a busy subreddit are not reachable at all.
4. **`crawlCommentsPerPost` multiplies cost.** 200 posts × default `maxCommentsPerPost` 200 can be 40,000 rows. Set `maxCommentsPerPost` deliberately.
5. **Search options don't apply to `startUrls`.** `searchSort`, `searchTime` and `withinCommunity` only shape `searchTerms` runs. For a subreddit URL, sort by putting it in the URL (`/r/x/new/`, `/r/x/top/?t=week`).
6. **`/s/` share links can't be scraped.** Resolve to the canonical `/comments/<id>/` permalink first.
7. **Disabling `fastMode` needs 2048 MB.** Leave it on unless exhaustive comment search inside one subreddit is the point.
8. **Dates are UTC, `YYYY-MM-DD`.** `postedAfter` is 00:00 UTC of that day, `postedBefore` is 23:59:59 UTC.
9. **NSFW is off by default.** Adult subreddits and posts need `includeNSFW: true`; NSFW communities also only appear in community search when it is on.
10. **`removedByCategory` is the only reliable removal signal.** Fields like `removed_by` or `banned_by` are moderator-only and null for everyone else.

## Common errors

| Error / symptom | Cause | Fix |
|---|---|---|
| `missing_targets` | No `searchTerms`, `startUrls` or `subredditUrls` (often an invented field name like `query`) | Use the real field names |
| `invalid_date_range` | `postedAfter` > `postedBefore`, or bad date format | Swap or fix to `YYYY-MM-DD` |
| `excessive_memory` warning | Run memory set above what fast mode needs | Harmless; leave default memory |
| 0 items, SUCCEEDED | Permalinks in `subredditUrls`, or literal search miss | See traps 1 and 2 |
| Run summary lists skipped targets | Private/banned/quarantined community, deleted user, or a 404 | Terminal; report it, don't retry |
| Slow run (>5 min) | Full subreddit + comments, or many `searchTerms` | Poll asynchronously; or split into smaller runs |

## Delivery to Slack / Notion / Sheets (MCP connectors)

`mcpConnector` names an MCP connector already authorized in the user's Apify account (Console → Integrations). `mcpTarget` is the destination ID (Slack channel ID, Notion database ID, Sheet ID). `mcpMode` is `perPost` (one message per post, capped by `mcpMaxItems`, default 50) or `summary` (one digest). `mcpMessage` is a template over any post field, e.g. `"**{{title}}** ({{score}}↑) {{postUrl}}"`. Delivery never fails the run; check the run log for connector errors.
