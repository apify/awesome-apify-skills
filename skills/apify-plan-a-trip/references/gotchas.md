# Gotchas: plan a trip (both Actors)

Cost, sequencing, and where to look for depth. This skill orchestrates the flights and hotels Actors, so most Actor-specific quirks live in the two sibling skills' references. This file covers what is specific to running them together.

## Cost guardrails (combined)

A whole trip is a handful of runs: one or two flight searches plus a lodging search per overnight stop. Each is pay per event.

- Flights: about $0.02 per page plus a $0.01 setup fee per run. Leave `fetch_booking_options` off unless the traveler wants booking links, because it bills per option.
- Hotels: about $0.018 per page plus a $0.02 setup fee per run. `max_pages: 1` per city and date is usually enough; never run `max_pages: 0` on a broad city query.

A typical 4-night, one-anchor trip (one round-trip flight search, three or four lodging searches) lands well under a dollar. Estimate and confirm before enabling booking options or running many pages.

## Sequencing

- Verify the anchor date before anything else; a wrong date invalidates every downstream search.
- Price flights before lodging: the arrival and departure days set the first and last lodging nights.
- Work out the driving before pricing the anchor-night lodging: the drive decides which city to search.
- Re-base and re-price if the availability guard trips (a small town selling out for the anchor night).

## Where to look for depth

- Flights inputs, `price_insights`, the parse routine, the booking-options cost: the `apify-plan-flights` skill and its `references/gotchas.md`.
- Hotels inputs, the availability guard, the string-versus-number quirks, the Airbnb routine, the subagent parse: the `apify-plan-hotel-stays` skill and its `references/gotchas.md` and `references/airbnb-and-parsing.md`.
- Trip judgment (anchoring, drive-time reasoning, sellouts, normalization, delivery): `references/trip-planning-lessons.md`.

## Schema quirks to remember when calling both

- Hotels `min_price`, `max_price`, and `guest_rating` are strings; flights `max_price` is an integer.
- Neither schema declares required fields; requiredness is enforced in code. Flights need `departure_id`, `arrival_id`, `outbound_date` (or `multi_city_json`); a hotels query search needs `q` and `check_in_date`.
- Flight durations are in minutes; hotel rates are per room for the party.
