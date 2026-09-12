# Gotchas: plan flights (johnvc/Google-Flights-Data-Scraper-Flight-and-Price-Search)

Cost guardrails, error recovery, input quirks, and the subagent parse routine. Read this on demand when building inputs, interpreting results, or when a run fails.

## Cost guardrails

Pricing model: pay per event. At the time of writing: about $0.02 per page of flight results processed (BRONZE tier), plus a $0.01 one-time setup fee per run, and a tiny per-result and per-start charge. Booking options are billed separately, about $0.02 per booking option returned, and only when `fetch_booking_options` is true. Confirm live prices with `apify actors info "johnvc/Google-Flights-Data-Scraper-Flight-and-Price-Search" --json 2>/dev/null` (look at pricing).

Estimate before running: a normal route and date pair is one page, around $0.03. The expensive switch is `fetch_booking_options: true`, which adds a per-option charge across many options; leave it off unless the traveler wants booking links, and warn before enabling it on a broad search.

Suggested confirmation thresholds:

- Rough estimate over $5: warn the traveler.
- Rough estimate over $20: get explicit confirmation before running.

## Common errors

| Error | Cause | Fix |
|-------|-------|-----|
| Empty or thin results | Bad airport code, or no service on the route and date | Verify IATA codes; try nearby airports with the comma-separated form (for example "EWR,JFK,LGA"). |
| Multi-city not working | Route and date fields set alongside `multi_city_json` | When you use `multi_city_json`, leave `departure_id`, `arrival_id`, and `outbound_date` empty; the JSON supplies each leg. |
| Prices look off for the market | Currency defaulted from country | Set `currency` explicitly. |
| No booking links in output | `fetch_booking_options` was false | Set it to true (and expect the per-option charge). |
| Only a few options returned | Google returned a short list, or `max_pages` capped it | Raise `max_pages` if you need deeper coverage. |

## Actor-specific notes

- `max_price` is an integer in this schema (the hotels Actor uses strings). Pass 600, not "600".
- From the dataset, the itinerary lives at the top level in `best_flights` (Google's curated shortlist), `other_flights` (the long tail), and a pre-flattened `all_flights` view; read `all_flights` for the quickest one-row-per-flight parse. A local `output_file` nests the same content under a `results` object.
- Number of stops is the length of the `layovers` array on each flight option. `total_duration` and each leg `duration` are in minutes.
- `type` is "Round trip" or "One way". For a round-trip the `flights` array holds all legs across both directions; group by direction when you display times.
- `price_insights` is the advice layer: `lowest_price`, `price_level` (low, typical, high), `typical_price_range`, and a `price_history` series. Use it to say whether now is a good time to book.
- The schema declares no required fields; requiredness is enforced in code (a one-way or round-trip search needs `departure_id`, `arrival_id`, and `outbound_date`).

## Parse results in a subagent

Large searches write minified JSON to a tool-results file rather than returning inline. To keep the main context clean, hand the saved file path to one subagent and ask it to:

1. Prefer the pre-flattened `all_flights` array: one row per flight, already carrying `airlines`, `route`, `stops`, `stops_label`, `price`, `duration`, `departure_time`, and `arrival_time`. Fall back to `best_flights` and `other_flights` when you need per-leg detail.
2. From `all_flights`, return one compact row per option: `airlines`, `route`, `stops_label`, `duration`, times, and `price`. From `best_flights` or `other_flights`, build the same by reading each leg's `airline`, counting `layovers` for stops, and converting `total_duration` minutes to hours and minutes.
3. Also return the `price_insights` summary (lowest price, level, typical range).
4. Verify every number against the data rather than estimating.

Feed those rows to `scripts/render_price_table.py`.
