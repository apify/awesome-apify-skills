---
name: apify-plan-hotel-stays
description: Plan hotel stays for a trip. Given one or more destinations, travel dates, party size, budget, and an optional rewards program, return a ranked, de-duplicated comparison of hotels and vacation rentals with nightly and total prices, ratings, review counts, and booking links, plus an email-ready HTML price table. Built on the Apify Google Hotels Search Scraper (johnvc/google-hotels-search-scraper). Use whenever someone wants to find a hotel, compare hotels in a city, decide where to stay, price lodging for specific dates, find Marriott or Hilton or Hyatt or IHG hotels for a stay, or asks "where should we stay" or "best hotel for these dates". It detects sold-out dates and offers to widen to a larger nearby city, normalizes per-night versus total price, and can price specific Airbnb listings too.
author: John Cole
author_url: https://github.com/johnisanerd
license: MIT
metadata:
  version: "1.0"
  keywords: "plan hotel stays, hotel search, compare hotels, where to stay, hotel prices, vacation rentals, rewards program, trip planning, google hotels, apify"
---

# Plan Hotel Stays: a ranked hotel and rental comparison with prices

Turn a destination and travel dates into a decision. This skill gathers the trip details, prices live hotels and vacation rentals across every location and date, ranks them, flags sold-out dates, respects a rewards program when the traveler has one, and hands back a comparison plus an email-ready price table people can actually book from.

## When to use this skill

- The traveler wants to find a hotel or compare hotels in a city for specific dates.
- They ask "where should we stay", "what is the best hotel for these dates", or "how much is lodging for this trip".
- They want to stay within a hotel loyalty program (Marriott, Hilton, Hyatt, IHG, Wyndham, and so on) and see which of that chain's properties fit.
- They want vacation rentals compared alongside hotels, or want a specific Airbnb priced for the dates.
- They want a shareable price table (Markdown or an email-ready HTML table) rather than a wall of search results.

Not for: making reservations (this reads prices, it does not book), flight search (use `apify-plan-flights`), or planning a whole multi-stop trip with logistics (use `apify-plan-a-trip`).

## Gather the trip details first, then map them to the search

Ask for what you are missing before searching. These map straight onto the Actor's inputs:

| Trip detail | Actor input |
|-------------|-------------|
| Destination(s) | `q` (for example "hotels near Denver International Airport", "hotels in Santa Fe, New Mexico") |
| Check-in and check-out dates | `check_in_date`, `check_out_date` (YYYY-MM-DD) |
| Party size and rooms | `adults`, `children`, `children_ages`; run per room if the group splits rooms |
| Budget range | `min_price`, `max_price` (pass as strings, for example "120") |
| Quality bar | `guest_rating` (string, for example "4.0"), `hotel_class` or `stars` |
| Airport vs city center | phrasing in `q` ("near the airport" vs "downtown") |
| Include rentals | `vacation_rentals: true` (a second run; see the workflow) |
| Rewards program | see the next section |
| Search locale | `gl`, `hl`, `currency` |

If dates come from a flight itinerary, use the arrival and departure days as the first and last nights.

## Rewards programs (search within a loyalty program)

The Actor has no loyalty filter, so approximate honestly with `scripts/rewards.py`:

```bash
python3 scripts/rewards.py "Marriott Bonvoy"
```

For a hotel program it returns the chain's brand family (Marriott returns Courtyard, Fairfield, SpringHill, Residence Inn, Westin, Sheraton, and the rest). Use that list two ways: brand-bias the query where it helps (`q: "Marriott hotels near Denver airport"`), and after the search keep or flag rows whose `name` contains one of the brand substrings. Be honest about the limit: this Actor reads public cash rates, not your loyalty account, and does not show points or award pricing. It tells the traveler which of their chain's properties exist and what they cost in cash, so they can book the award stay through their own program.

## What you get back (one object per results page)

Each dataset item is one page of results (about 18 to 20 properties). Each property in the `properties` array carries `name`, `type` (hotel or vacation rental), `rate_per_night` and `total_rate` (a display string `lowest` plus a numeric `extracted_lowest`), `overall_rating`, `reviews` count, `amenities`, `gps_coordinates`, `link` (booking URL), and `property_token` (the dedupe key, and the input for photos or reviews). When present: `hotel_class` and `extracted_hotel_class`, `deal`, a per-vendor `prices` array, and `reviews_breakdown`.

## Prerequisites

- Apify account (sign up at https://apify.com?fpr=9n7kx3&fp_sid=awesomeskills).
- Authenticate with `apify login`, or set an `APIFY_TOKEN` environment variable (Apify Console, Settings, Integrations).
- Optional: the Chrome MCP tools, only if you want to price specific Airbnb listings.

## The Actor

- Store page: https://apify.com/johnvc/google-hotels-search-scraper?fpr=9n7kx3&fp_sid=awesomeskills
- Actor ID: `johnvc/google-hotels-search-scraper`
- Pricing: pay per page of results processed plus a small per-run setup fee (see `references/gotchas.md`).

## Run it with the Apify CLI

One search for a city and stay date:

```bash
apify actors call "johnvc/google-hotels-search-scraper" -i '{"search_type":"search","q":"hotels near Denver International Airport","check_in_date":"2026-10-15","check_out_date":"2026-10-16","adults":3,"currency":"USD","max_pages":1}' \
  --json \
  --user-agent apify-awesome-skills/apify-plan-hotel-stays \
  2>/dev/null
```

Filtered: 4-star and up, guest rating at least 4.0, under 200 USD a night:

```bash
apify actors call "johnvc/google-hotels-search-scraper" -i '{"search_type":"search","q":"hotels in Santa Fe, New Mexico","check_in_date":"2026-10-16","check_out_date":"2026-10-18","adults":3,"hotel_class":"4,5","guest_rating":"4.0","max_price":"200","currency":"USD","max_pages":1}' \
  --json \
  --user-agent apify-awesome-skills/apify-plan-hotel-stays \
  2>/dev/null
```

Include vacation rentals (a separate run):

```bash
apify actors call "johnvc/google-hotels-search-scraper" -i '{"search_type":"search","q":"vacation rentals in Albuquerque, New Mexico","check_in_date":"2026-10-16","check_out_date":"2026-10-18","adults":3,"vacation_rentals":true,"max_pages":1}' \
  --json \
  --user-agent apify-awesome-skills/apify-plan-hotel-stays \
  2>/dev/null
```

Every call carries the three flags this repo expects: `--json`, `--user-agent apify-awesome-skills/apify-plan-hotel-stays`, and `2>/dev/null`.

## Run it from Claude or another AI agent (MCP)

The Actor is MCP-ready. Add the hosted server URL:

`https://mcp.apify.com/?tools=actors,docs,johnvc/google-hotels-search-scraper`

Then ask, for example: "Price hotels near Denver airport for Oct 15 to 16 for 3 adults, and the ten best-rated four-star options under 200 dollars a night." MCP setup docs: https://docs.apify.com/platform/integrations/mcp

## Workflow

1. Gather and confirm the trip details above. Fill gaps by asking, not guessing.
2. Map each location and date pair to one search input. Remember the schema quirks: `min_price`, `max_price`, and `guest_rating` are strings, not numbers. Requiredness is enforced in code, not the schema; a query search needs `q` plus `check_in_date`.
3. Run one search per location and date. Add one `vacation_rentals: true` run per location if rentals are wanted (rentals and `hotel_class` are not compatible; drop the class filter on the rentals run).
4. Availability guard. A property with no `rate_per_night` for the date is not available, not free. If most well-rated properties in a small town come back with no rate, say so and offer to widen the search to a larger nearby city (Socorro sold out for an event night points you to Albuquerque). See `references/gotchas.md`.
5. Parse in a subagent. A single search is 50 to 85 KB of minified JSON that lands in a tool-results file, not inline; `properties` is an array, so field projection with `fields="properties.name"` returns empty objects (request `fields="properties"` or the whole item). Hand the saved file paths to one subagent that extracts compact rows (name, class, rating, reviews, per-night, total, coords, link, property_token) and verifies the numbers. This keeps hundreds of KB of raw JSON out of the main context.
6. Normalize and rank. For a one-night stay `total_rate` equals `rate_per_night`. Hotel rates are per room for the party; rental totals fold in cleaning and service fees, which make a single-night rental look expensive per night, so label the basis (per night vs total) in the output. Rank on rating and review count and price. Show `hotel_class` but do not rank on it; it is occasionally wrong.
7. Apply the rewards filter or flag if a program was given.
8. Optional Airbnb. Airbnb is never in Google Hotels results, even with `vacation_rentals: true`. If the traveler supplies Airbnb room URLs or asks for Airbnb, price each with the Chrome MCP routine in `references/airbnb-and-parsing.md`.
9. Render the deliverables. Build the row set and run `scripts/render_price_table.py` to produce both a Markdown table and an email-ready HTML table with booking links and a "verify before booking" footer.

## Inputs (Actor parameters)

- `search_type` (enum: `search`, `autocomplete`, `photos`, `reviews`; default `search`)
- `q` (destination or hotel query; required for a query search)
- `check_in_date` / `check_out_date` (YYYY-MM-DD)
- `adults` (default 2), `children` (default 0), `children_ages` (required if children above 0)
- `min_price`, `max_price`, `currency` (prices are strings)
- `stars`, `hotel_class` (2 to 5), `guest_rating` (string), `amenities`, `property_type` (filters)
- `vacation_rentals` (boolean) plus `rental_type`, `bedrooms`, `bathrooms`
- `gl`, `hl` (country and language)
- `max_pages` (default 1, 0 = all pages; the cost cap)
- `property_token` (single-property mode; also required for photos and reviews)

## Cost guardrails

Pay per event: a per-run setup fee plus a fee per page of results processed. A one-page search lands around $0.04. `max_pages` is the hard cost cap; never run `max_pages: 0` on a broad city query without an estimate. Full thresholds are in `references/gotchas.md`.

## Honest limits

- Read-only. The Actor reads prices; it does not make reservations.
- Cash rates only, not points or award pricing, and not your loyalty account balance.
- One dataset item per results page, not per hotel; flatten the `properties` array.
- Airbnb is not in Google Hotels; price it separately (see references).
- Star class is noisy; use it as a label, not a ranking key.

## Troubleshooting

See `references/gotchas.md` for empty results, the children-ages validation error, the rentals-plus-class incompatibility, currency defaults, and the availability guard. See `references/airbnb-and-parsing.md` for the subagent parse routine and the Airbnb scraping routine. See `references/actor-index.md` for the Actor routing table.

## Bundled scripts

- `scripts/render_price_table.py`: rows in, a Markdown table and an email-ready HTML table out.
- `scripts/rewards.py`: a rewards program name in, airline codes or hotel brand families out.

## Related travel skills and Actors

- Plan flights for the same trip: the `apify-plan-flights` skill, built on https://apify.com/johnvc/Google-Flights-Data-Scraper-Flight-and-Price-Search?fpr=9n7kx3&fp_sid=awesomeskills
- Plan the whole trip (flights, lodging, and logistics): the `apify-plan-a-trip` skill.
- Export raw hotel data as JSON: the `apify-scrape-hotel-prices` skill.
- Destination discovery: https://apify.com/johnvc/google-travel-explore-api?fpr=9n7kx3&fp_sid=awesomeskills
- Restaurants and attractions near a property: https://apify.com/johnvc/google-local-api?fpr=9n7kx3&fp_sid=awesomeskills
