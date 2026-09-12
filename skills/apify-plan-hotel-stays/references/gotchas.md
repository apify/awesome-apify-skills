# Gotchas: plan hotel stays (johnvc/google-hotels-search-scraper)

Cost guardrails, the availability guard, error recovery, and input quirks. Read this on demand when building inputs, interpreting results, or when a run fails.

## Cost guardrails

Pricing model: pay per event. At the time of writing: about $0.018 per page of hotel search results processed (BRONZE tier), plus a $0.02 one-time setup fee per run, $0.00001 per dataset result, and a tiny per-start charge. Prices differ by usage tier; confirm the live price on the Store card or with `apify actors info "johnvc/google-hotels-search-scraper" --json 2>/dev/null` (look at pricing).

Estimate before running: cost is roughly the setup fee plus `max_pages` times the per-page price. One page carries about 20 properties.

- 1 city, 1 page: about $0.04.
- 5 cities, 1 page each: about $0.12.
- 5 cities, 5 pages each: about $0.55.

Suggested confirmation thresholds:

- Rough estimate over $5: warn the traveler.
- Rough estimate over $20: get explicit confirmation before running.
- Always present cost as "around $X", not a guarantee.

`max_pages` is the hard cost cap. The dangerous value is 0: it means fetch every available page, and a broad city query can report thousands of results. Never run `max_pages: 0` on a city-level query without an explicit go-ahead. For most trip planning, 1 page per city and date is plenty; the top ~20 results cover the bookable set.

## The availability guard (a signal, not an error)

Many properties return no `rate_per_night` for a given date. That means no availability for those dates, not a price of $0. Treat it as a signal:

1. When you flatten results, separate "priced" from "no rate returned".
2. If a large share of the well-rated properties in a small town come back with no rate, the town is likely selling out (common around events). Say so plainly.
3. Offer to widen the search to a larger nearby city and re-run. Example: Socorro, New Mexico sold out for a Trinity Site open-house night while only motels remained, so the search widened to Albuquerque, about an hour north.

## Common errors

| Error | Cause | Fix |
|-------|-------|-----|
| Empty `properties` array | Query too narrow, filters too tight, or dates invalid | Broaden the query, drop filters, check `check_in_date` is in the future and `check_out_date` is after it. |
| Validation error on children | `children` above 0 without `children_ages` | Pass `children_ages` as a comma-separated string, for example "5,8". |
| No results with class filter on rentals | `hotel_class` combined with `vacation_rentals: true` | The schema marks them incompatible; drop the class filter on the rentals run. |
| Prices look wrong for the market | Currency defaulted from country | Set `currency` explicitly (ISO 4217, like "USD" or "EUR"). |
| Fewer pages than expected | Google had fewer results, or `max_pages` capped it | Check `search_metadata.pagination_limit_reached`; raise `max_pages` if true. |

## Actor-specific notes

- One dataset item per results page, not per hotel. Flatten `properties` into rows client-side.
- Compute on the numeric fields (`rate_per_night.extracted_lowest`, `total_rate.extracted_lowest`, `extracted_hotel_class`); the unprefixed twins are display strings like "$171" or "4-star hotel".
- For a one-night stay `total_rate` equals `rate_per_night`. For longer stays compare on total, then show per-night for context. Hotel rates are per room for the party; rental totals include cleaning and service fees, so a single-night rental looks expensive per night. Label the basis.
- Star class (`extracted_hotel_class`) is occasionally wrong (a budget brand showing 5 stars). Show it, do not rank on it. Rank on rating, review count, and price.
- `property_token` is the stable dedupe key across runs and the input for `reviews`, `photos`, and single-property detail pulls. It is marked secret in the schema, so the Console masks it; passing it via API or CLI works normally.
- `min_price`, `max_price`, and `guest_rating` are strings in the schema, for example "120" and "4.0". The schema declares no required fields; requiredness is enforced in code (a query search needs `q` and `check_in_date`).
