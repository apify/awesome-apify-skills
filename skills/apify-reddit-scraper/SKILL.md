---
name: apify-reddit-scraper
description: >
  Scrape Reddit with the harshmaur/reddit-scraper Apify Actor: keyword search across all of Reddit
  or inside one subreddit, full subreddit listings, post permalinks with complete comment threads,
  user profiles and community info, with date-range filters and optional delivery of results into
  Slack, Notion, Google Sheets or Airtable through Apify MCP connectors. Use when the user says
  "monitor my brand on Reddit", "find leads on Reddit", "what is Reddit saying about X", "scrape
  r/<subreddit>", "pull all comments from this Reddit thread", "Reddit posts about X since <date>",
  "track competitor mentions on Reddit", "Reddit sentiment for X", "build a Reddit dataset",
  "scrape a Reddit user's history", or any request for Reddit posts, comments, subreddits or users
  as structured data. No Reddit API key or login is needed. Out of scope: posting or replying on
  Reddit, private or quarantined communities, and semantic/topic search (Reddit search is literal).
author: Harsh Maur
author_url: https://github.com/harshmaur
metadata:
  category: data-extraction
  keywords: "reddit, reddit-scraper, subreddit, reddit-comments, brand-monitoring, social-listening, lead-generation, reddit-search, reddit-sentiment, reddit-dataset, community-research, mcp-connector, slack, notion"
---

# Reddit Scraper

Turn a plain-language Reddit request into one run of `harshmaur/reddit-scraper` and hand back structured posts, comments, users or communities. One Actor covers search, subreddit listings, permalinks, user profiles and community metadata, so routing is about picking the right **input field**, not the right Actor.

Disclosure: the Actor this skill routes to is paid (pay per result) and built by the skill author. Links carry no affiliate parameters.

## Example prompts

Prompts this skill handles:

- "Find every Reddit post from the last 30 days where someone asks for a Notion alternative, and pull the comments."
- "Scrape the newest 200 posts in r/SaaS and r/startups."
- "Give me all comments on this thread: https://www.reddit.com/r/webdev/comments/abc123/..."
- "Track mentions of `Linear` and `Jira` on Reddit since 2026-08-01 and send each hit to my Slack channel."
- "What has u/spez posted recently?"

Out of scope (the boundary):

- "Post a reply in r/startups" — this Actor only reads Reddit. Use the Reddit API with the user's own account.
- "Find Reddit threads *about* the pain of managing invoices" — Reddit search is literal keyword matching, not semantic. Expand the topic into 3–8 short literal keywords first (see Step 1), then run one search per keyword.

## Prerequisites

- Apify account ([sign up](https://apify.com)); the free plan covers small runs.
- Authentication via one of:
  - `apify login` (OAuth, if using the Apify CLI)
  - `APIFY_TOKEN` environment variable
  - Token from [Apify Console → Settings → Integrations](https://console.apify.com/settings/integrations)
- Apify CLI (`npm install -g apify-cli`), or the Apify MCP server (see "Calling the Actor").

## Workflow

### Step 1 — Classify the request and pick input fields

| User gives you | Field to use | Notes |
|---|---|---|
| Keywords, a brand, a product, a phrase | `searchTerms` (array) | One search per term. Keep terms short and literal (`"notion alternative"`, not a sentence). Free plans search the first 40 terms per run. |
| Keywords + "only in r/X" | `searchTerms` + `withinCommunity` | Accepts `developers`, `r/developers` or the full subreddit URL. |
| A subreddit to scrape in bulk | `subredditUrls` (array of names or community URLs) | Fetches far more posts than a plain listing. **Never put post permalinks here** — they are dropped silently and the run finishes with 0 items. |
| Post, user-profile, subreddit or search-page URLs | `startUrls` (array of `{ "url": ... }`) | Search options (sort/time/community) do NOT apply to these. |
| "since <date>" / "between <dates>" | `postedAfter`, `postedBefore` (`YYYY-MM-DD`) | Also `commentedAfter` / `commentedBefore` for comments. Results are fetched newest-first and stop early once older than the window. |
| "with comments" / "full thread" | `crawlCommentsPerPost: true` + `maxCommentsPerPost` | Multiplies result count and cost — see gotchas. |
| "comments that mention X" | `searchComments: true`, `searchPosts: false` | Comment search across Reddit. |
| "which subreddits are about X" | `searchCommunities: true` | Returns `dataType: "community"` rows. |
| "send to Slack / Notion / Sheets" | `mcpConnector` + `mcpTarget` (+ `mcpMessage`) | Requires an authorized MCP connector in the user's Apify account; see `references/input-mapping.md`. |

When the user's phrase is descriptive rather than literal ("people frustrated with their CRM"), draft 3–8 literal keywords (`"CRM sucks"`, `"switching from HubSpot"`, `"CRM recommendation"`), show them, and run them as `searchTerms`.

### Step 2 — Build the input

Start from these defaults and change only what the request needs:

```json
{
  "searchTerms": ["notion alternative"],
  "searchPosts": true,
  "searchComments": false,
  "searchCommunities": false,
  "searchSort": "new",
  "searchTime": "month",
  "maxPostsCount": 50,
  "crawlCommentsPerPost": false,
  "maxCommentsPerPost": 50,
  "includeNSFW": false,
  "fastMode": true
}
```

- `searchSort`: `new` for monitoring, `relevance` for precise matching, `top` / `hot` for what performed, `comments` for the most discussed.
- `searchTime`: `hour` | `day` | `week` | `month` | `year` | `all`. Prefer `postedAfter`/`postedBefore` when the user names dates.
- `maxPostsCount` is the **total** across all inputs, max 50,000. Reddit itself caps any single listing or search at roughly 1,000 posts; for more history split by date windows or by subreddit.
- Leave `fastMode: true` and the default memory. Turning fast mode off needs 2048 MB and is only worth it for exhaustive subreddit comment search.
- Do not pass `proxy`; the Actor manages its own proxying.

Fetch the live schema if a field is in doubt:

```bash
apify actors info "harshmaur/reddit-scraper" --input --json \
  --user-agent apify-awesome-skills/apify-reddit-scraper 2>/dev/null
```

### Step 3 — Estimate cost, then run

Pricing is pay per result: **$0.02 per run start + $0.002 per stored item** ($2.00 per 1,000; $1.50 per 1,000 on plans with Store discounts). Estimate items = posts + comments before running. Under ~2,500 items (about $5) just run it; above that, state the estimate and get a yes. `crawlCommentsPerPost` is the usual surprise: 200 posts × 50 comments = 10,000 comments ≈ $20.

```bash
apify actors call "harshmaur/reddit-scraper" \
  -i '{"searchTerms":["notion alternative"],"searchTime":"month","maxPostsCount":50}' \
  --json \
  --user-agent apify-awesome-skills/apify-reddit-scraper \
  2>/dev/null
```

The JSON output reports `run.status` and the dataset id at `storage.defaultDatasetId`. For small runs you can add `-o` (`--output-dataset`) to print the items directly instead of fetching them in Step 4. Typical runs finish in 20–90 s; a full subreddit with comments can take several minutes, so run those asynchronously and poll if the agent has a time budget.

### Step 4 — Fetch and deliver

```bash
apify datasets get-items "DATASET_ID" --format json \
  --user-agent apify-awesome-skills/apify-reddit-scraper \
  2>/dev/null > /tmp/reddit.json
```

Every row carries `dataType` (`post` | `comment` | `community` | `user`). Key fields:

| `dataType` | Fields you will use |
|---|---|
| `post` | `title`, `body`, `postUrl`, `communityName`, `authorName`, `score`, `commentsCount`, `createdAt`, `flair`, `searchTerm`, `removedByCategory` (why a `[removed]` post was pulled: `moderator`, `automod_filtered`, `reddit`, `author`, `deleted`) |
| `comment` | `body`, `url`, `postId`, `postTitle`, `authorName`, `score`, `commentCreatedAt`, `parentId`, `depth` |
| `community` | `name`, `title`, `description`, `membersCount`, `onlineUsersCount`, `nsfw`, `rules` |
| `user` | `username`, `totalKarma`, `bio`, `followersCount`, `createdAt` |

Report back: what was searched (exact input), item count by `dataType`, whether a limit or date window truncated the run, a handful of representative rows with `postUrl` links, and the next parameter change if the set was too broad or too thin. Group by `communityName` or `searchTerm` when several were used. Never assert sentiment from titles alone; quote the `body`.

### Optional — recurring monitoring

If the user re-runs the same search on a cadence, propose an Apify Schedule (Console → Schedules) with `postedAfter` set relative to the last run, and `mcpConnector` delivery so new hits land in Slack, Notion, Sheets or Airtable without an agent in the loop.

## Actor routing

| User need | Actor ID | Tier | Best for |
|-----------|----------|------|----------|
| Posts, comments, users, communities, search, date windows, MCP delivery | `harshmaur/reddit-scraper` | community | Everything in this skill; single Actor, single input schema |

Sibling listings by the same author exist for narrower jobs (`harshmaur/reddit-comments-scraper`, `harshmaur/reddit-search-scraper`, `harshmaur/reddit-subreddit-scraper`, `harshmaur/reddit-user-scraper`). They run the same engine with a trimmed input, so prefer the main Actor unless the user asks for one of them by name.

## Calling the Actor — choose your interface

### Option A: Apify CLI (recommended for portability)

Every command carries the three flags shown above: `--json` (or `--format json` for `datasets get-items`), `--user-agent apify-awesome-skills/apify-reddit-scraper`, and `2>/dev/null`.

### Option B: Apify MCP server

Point any MCP client at `https://mcp.apify.com/?tools=harshmaur/reddit-scraper` (bearer `APIFY_TOKEN`, or OAuth). The Actor appears as a callable tool; pass the same input JSON. Docs: <https://docs.apify.com/platform/integrations/mcp>.

### Option C: MCP client of your choice (e.g. `mcpc`)

See <https://github.com/apify/mcpc>.

## Troubleshooting

- **0 items, run SUCCEEDED, no error** → post permalinks were put in `subredditUrls`. Move them to `startUrls`.
- **Empty search results** → the term was a sentence. Reddit search is literal; shorten to 1–3 words, drop `searchTime` to `all`, or remove `withinCommunity`.
- **Fewer posts than requested on a big subreddit** → Reddit's ~1,000-per-listing ceiling. Split with `postedAfter`/`postedBefore` windows or add `subredditUrls` per community.
- **`invalid_date_range`** → `postedAfter` is later than `postedBefore`, or the format isn't `YYYY-MM-DD`.
- **`missing_targets`** → none of `searchTerms`, `startUrls`, `subredditUrls` was set (a common cause is using an invented field like `query` or `keywords`).
- **`/s/` share links** (`reddit.com/r/x/s/abc`) → resolve them in a browser to the full permalink first; the short form cannot be scraped.
- **Private, banned or quarantined subreddit** → terminal skip, noted in the run summary; nothing to retry.
- **Cost higher than expected** → `crawlCommentsPerPost` was on. Re-run with it off, or lower `maxCommentsPerPost`.
- For cost tables and recovery flows, see [references/gotchas.md](references/gotchas.md); for the full field mapping, [references/input-mapping.md](references/input-mapping.md).
