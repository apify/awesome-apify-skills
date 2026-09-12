---
name: apify-plan-flights
description: Plan flights for a trip. Given an origin and destination (or several), travel dates, party size, budget, and an optional airline or rewards program, return ranked flight options with price, number of stops, airlines, total duration, and layovers, plus a comparison table. Built on the Apify Google Flights Data Scraper (johnvc/Google-Flights-Data-Scraper-Flight-and-Price-Search). Use whenever someone wants to find flights, compare airfare, price a route for specific dates, search one-way, round-trip, or multi-city itineraries, fly within an airline alliance or miles program (United, American, Delta, Alaska, and partners), or asks "how much are flights" or "what is the cheapest flight". It reads Google Flights price insights so you can say whether a fare is a good deal, and can fetch booking links on request.
author: John Cole
author_url: https://github.com/johnisanerd
license: MIT
metadata:
  version: "1.0"
  keywords: "plan flights, flight search, compare airfare, cheapest flight, round trip flights, multi-city, airline miles, price insights, trip planning, google flights, apify"
---

# Plan Flights: ranked flight options with prices and stops

Turn a route and dates into a shortlist. This skill gathers the trip details, prices live flights for one-way, round-trip, or multi-city itineraries, ranks them by price and convenience, respects an airline or miles program when the traveler has one, reads Google Flights price insights so you can say if a fare is high or low, and hands back a clean comparison table.

## When to use this skill

- The traveler wants to find flights or compare airfare for a route and dates.
- They ask "how much are flights", "what is the cheapest flight", or "what are my options" for a trip.
- They want to fly within an alliance or miles program (United and Star Alliance, American and Oneworld, Delta and SkyTeam, Alaska and partners).
- They want a round-trip, one-way, or multi-city itinerary priced.
- They want to know whether a fare is a good deal right now (price insights).

Not for: booking tickets (this reads prices, it does not book), hotel search (use `apify-plan-hotel-stays`), or planning a whole multi-stop trip with lodging and logistics (use `apify-plan-a-trip`).

## Gather the trip details first, then map them to the search

| Trip detail | Actor input |
|-------------|-------------|
| Origin and destination airports | `departure_id`, `arrival_id` (IATA codes; comma-separate to search several, for example "CDG,ORY") |
| Outbound and return dates | `outbound_date`, `return_date` (YYYY-MM-DD; omit return for one-way) |
| Multi-city itinerary | `multi_city_json` (replaces the three fields above) |
| Party size | `adults`, `children`, `infants` |
| Budget ceiling | `max_price` (an integer here, for example 600) |
| Nonstop preference | `max_stops` (0 = nonstop only, 1 = one stop max) |
| Airline or rewards program | `airlines` (CSV of codes); see the next section |
| Cabin quality | `exclude_basic` (drop basic economy) |
| Booking links | `fetch_booking_options` (bills per option; see cost) |
| Search locale | `gl`, `hl`, `currency` |

If the traveler gives cities rather than airports, resolve to IATA codes first (and offer nearby-airport alternatives via the comma-separated form).

## Rewards programs (fly within a miles program)

Use `scripts/rewards.py` to turn a program into an `airlines` filter:

```bash
python3 scripts/rewards.py "United MileagePlus"
```

For an airline program it returns the airline plus its alliance partner codes (United returns the Star Alliance set). Pass those as the `airlines` input so results stay bookable and creditable to that program. Southwest and JetBlue return just their own code (no alliance). Honest limit: this Actor reads cash fares, not award (miles) pricing or your account balance. It shows which flights on your program's airlines exist and what they cost in cash, so you can then price the award seat in your own program.

## What you get back

The Apify dataset item carries, at the top level, `best_flights` (Google's curated shortlist), `other_flights` (the long tail), a pre-flattened `all_flights` view (one row per flight, the easiest to parse), `price_insights`, and `airports`. Each option in `best_flights` and `other_flights` carries `flights` (the legs), `layovers`, `total_duration` (minutes), `carbon_emissions`, `price` (a number), `type` (Round trip or One way), and a `departure_token`. Each leg carries `departure_airport`, `arrival_airport`, `duration`, `airline`, `flight_number`, `travel_class`, `airplane`, and `legroom`. The `all_flights` rows carry `airlines`, `route`, `stops`, `stops_label`, `price`, `duration`, `departure_time`, and `arrival_time` already flattened. `price_insights` carries `lowest_price`, `price_level` (low, typical, high), `typical_price_range`, and `price_history`. Booking links appear in `booking_options` only when `fetch_booking_options` is true. Note: when the Actor saves a local JSON file via `output_file`, the same content is nested under a `results` object; from the dataset (API, CLI, or MCP) it is at the top level.

## Prerequisites

- Apify account (sign up at https://apify.com?fpr=9n7kx3&fp_sid=awesomeskills).
- Authenticate with `apify login`, or set an `APIFY_TOKEN` environment variable (Apify Console, Settings, Integrations).

## The Actor

- Store page: https://apify.com/johnvc/Google-Flights-Data-Scraper-Flight-and-Price-Search?fpr=9n7kx3&fp_sid=awesomeskills
- Actor ID: `johnvc/Google-Flights-Data-Scraper-Flight-and-Price-Search`
- Pricing: pay per page of results processed plus a small per-run setup fee; booking options bill separately (see `references/gotchas.md`).

## Run it with the Apify CLI

Round-trip search for 3 adults:

```bash
apify actors call "johnvc/Google-Flights-Data-Scraper-Flight-and-Price-Search" -i '{"departure_id":"IAD","arrival_id":"DEN","outbound_date":"2026-10-15","return_date":"2026-10-19","adults":3,"currency":"USD","max_pages":1}' \
  --json \
  --user-agent apify-awesome-skills/apify-plan-flights \
  2>/dev/null
```

Nonstop only, on a miles program, under a budget:

```bash
apify actors call "johnvc/Google-Flights-Data-Scraper-Flight-and-Price-Search" -i '{"departure_id":"IAD","arrival_id":"DEN","outbound_date":"2026-10-15","return_date":"2026-10-19","adults":3,"max_stops":0,"airlines":"UA,AC,LH","max_price":600,"exclude_basic":true,"currency":"USD"}' \
  --json \
  --user-agent apify-awesome-skills/apify-plan-flights \
  2>/dev/null
```

Every call carries the three flags this repo expects: `--json`, `--user-agent apify-awesome-skills/apify-plan-flights`, and `2>/dev/null`.

## Run it from Claude or another AI agent (MCP)

The Actor is MCP-ready. Add the hosted server URL:

`https://mcp.apify.com/?tools=actors,docs,johnvc/Google-Flights-Data-Scraper-Flight-and-Price-Search`

Then ask, for example: "Price round-trip flights IAD to DEN Oct 15 to 19 for 3 adults, nonstop preferred, and tell me if the fare is a good deal." MCP setup docs: https://docs.apify.com/platform/integrations/mcp

## Workflow

1. Gather and confirm the trip details above. Resolve cities to IATA codes.
2. Map to the search input. Remember the quirk: `max_price` is an integer here (the hotels Actor uses strings). The schema declares no required fields; a one-way or round-trip search needs `departure_id`, `arrival_id`, and `outbound_date`, or use `multi_city_json` instead.
3. Run the search. Keep `max_pages` at 1 for a normal trip; raise it only when you need deep coverage.
4. Parse in a subagent when the result is large. Prefer the pre-flattened `all_flights` array (one row per flight, with `airlines`, `route`, `stops_label`, `price`, `duration`, and times already extracted); fall back to `best_flights` and `other_flights` (count `layovers` for stops, read each leg's `airline`, convert `total_duration` minutes to hours and minutes). Verify numbers against the data. See `references/gotchas.md`.
5. Rank by price first, then by stops, then by total duration. Surface the best nonstop and the cheapest overall separately when they differ.
6. Read `price_insights`. Tell the traveler whether the current `lowest_price` sits below, inside, or above the `typical_price_range`, and mention the `price_level`. This is what turns a list into advice.
7. Apply the airlines filter for a rewards program (step done at input time via `airlines`).
8. Booking links, only if asked. Set `fetch_booking_options: true`; it makes extra requests and bills per booking option, so confirm first.
9. Render the deliverable. Build the row set and run `scripts/render_price_table.py` for a Markdown table and an email-ready HTML table.

## Inputs (Actor parameters)

- `departure_id`, `arrival_id` (IATA codes; comma-separated for multiple)
- `outbound_date`, `return_date` (YYYY-MM-DD; omit return for one-way)
- `multi_city_json` (JSON string; replaces the route and date fields)
- `adults` (default 1), `children` (default 0), `infants` (default 0)
- `max_price` (integer), `max_stops` (0 = nonstop), `airlines` (CSV codes)
- `exclude_basic` (boolean), `fetch_booking_options` (boolean; bills per option)
- `gl`, `hl` (country and language enums), `currency`
- `max_pages` (default 1, 0 = no limit)

## Cost guardrails

Pay per event: a per-run setup fee plus a fee per page of results processed (around $0.02 per page at BRONZE tier). Booking options are billed per option returned, so leave `fetch_booking_options` off unless the traveler wants links. Full thresholds are in `references/gotchas.md`.

## Honest limits

- Read-only. The Actor reads fares; it does not book tickets.
- Cash fares only, not award or miles pricing, and not your account balance.
- `total_duration` and leg `duration` are minutes; convert for display.
- Number of stops is the length of the `layovers` array, not a separate field.
- Fares move; a quote is for the moment it was pulled.

## Troubleshooting

See `references/gotchas.md` for empty results, airport-code issues, the multi-city format, the booking-options cost, and the subagent parse routine. See `references/actor-index.md` for the Actor routing table.

## Bundled scripts

- `scripts/render_price_table.py`: rows in, a Markdown table and an email-ready HTML table out.
- `scripts/rewards.py`: a rewards program name in, airline codes or hotel brand families out.

## Related travel skills and Actors

- Plan lodging for the same trip: the `apify-plan-hotel-stays` skill, built on https://apify.com/johnvc/google-hotels-search-scraper?fpr=9n7kx3&fp_sid=awesomeskills
- Plan the whole trip (flights, lodging, and logistics): the `apify-plan-a-trip` skill.
- Use the flights data as a raw API or dataset: the `apify-google-flights-api` and `apify-flight-price-api` skills.
- Destination discovery: https://apify.com/johnvc/google-travel-explore-api?fpr=9n7kx3&fp_sid=awesomeskills
