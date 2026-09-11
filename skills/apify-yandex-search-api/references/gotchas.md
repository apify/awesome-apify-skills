# Gotchas: Yandex search API (johnvc/Scrape-Yandex)

Cost guardrails, error recovery, and input quirks. The agent reads this on demand when building inputs or when a run fails.

## Cost guardrails

Pricing model: pay per event. Read the current `pricingInfos` from `apify api GET /v2/acts/johnvc~Scrape-Yandex -H 'User-Agent: apify-awesome-skills/apify-yandex-search-api'` before estimating. Use the latest pricing entry whose `startedAt` is not in the future and the tier of the authenticated account. `apify actors info --json` can truncate this Actor's metadata, so verify that the response parses before using it.

Estimate the run from the one-time `setup` event, the number of `page_processed` events, and `apify-default-dataset-item` for each stored dataset row. Read all current event rates; a nested result is not itself a dataset row. Use a positive `max_pages` limit; zero means no limit.

Suggested confirmation thresholds:

- Rough estimate over $5: warn the user.
- Rough estimate over $20: get explicit confirmation before running.
- Always present cost as "around $X", not a guarantee.

## Common errors

| Error | Cause | Fix |
|-------|-------|-----|
| Empty dataset | Query has no results on that domain or region | Broaden the query; check it manually on the chosen `yandex_domain`. |
| Results in the wrong language | `lr` set without `lang` and domain | Set `yandex_domain`, `lang`, and `lr` together. |
| Fewer pages than `max_pages` | Yandex ran out of results | Expected; check `pagination_limit_reached` in run metadata. |
| Error rows with `error_type` | Transient fetch failure on one page | Inspect the original run and dataset first; retry failed pages only after confirming the run is terminal and checking its charges. Other pages may already have returned. |

## Actor-specific notes

- `text` is the only required input; organic results are on by default.
- Output shape: one dataset item per page per result type, tagged `item_type` (`organic`, `ads`, `knowledge_graph`, `inline_images`, `inline_videos`). The actual results are nested arrays on the item (`organic_results`, `ads_results`, and so on): flatten client-side for CSV or row-level work.
- Organic rows always carry `position`, `title`, `link`, `displayed_link`, `snippet`; `date`, `rich_snippet`, and `sitelinks` appear only when Yandex shows them.
- `lr` region IDs: 213 Moscow, 2 Saint Petersburg, 65 Novosibirsk; any of 123,000 plus location IDs work.
- `sort_mode` "date" plus `period` gives a fresh-results read; default is relevance.
- The Yandex image search API skill covers `include_image_search`; dedicated video search is outside these two skills.
- Ongoing rank tracking requires scheduled runs, stored snapshots, and comparison; an alternative billing edition alone does not provide that workflow.
