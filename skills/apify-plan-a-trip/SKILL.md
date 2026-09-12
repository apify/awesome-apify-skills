---
name: apify-plan-a-trip
description: Plan an entire trip end to end, flights and hotels and logistics. Given an origin, a destination or a fixed anchor event, rough dates, party size, budget, and optional airline and hotel rewards programs, verify the anchor date live, price flights, price lodging at each overnight stop, reason about drive times and whether a leg is a day trip or an overnight, detect sold-out towns and re-base to a larger nearby city, and produce an itinerary plus a total-cost price table. Built on the Apify Google Flights Data Scraper and Google Hotels Search Scraper (johnvc/Google-Flights-Data-Scraper-Flight-and-Price-Search and johnvc/google-hotels-search-scraper), and reuses the plan-flights and plan-hotel-stays workflows. Use whenever someone says "plan a trip to", "help me plan an event trip", "flights and hotels for", "itinerary and budget for these dates", or describes a trip anchored to a game, festival, wedding, conference, or open house.
author: John Cole
author_url: https://github.com/johnisanerd
license: MIT
metadata:
  version: "1.0"
  keywords: "plan a trip, trip planner, flights and hotels, itinerary, event trip, travel budget, drive time planning, trip planning, google flights, google hotels, apify"
---

# Plan a Trip: flights, hotels, and logistics into one itinerary

Take a trip from a rough idea to a bookable plan. This skill anchors on the fixed thing (an event, or a destination and dates), verifies it against today's facts, prices the flights, works out the driving and where to sleep each night, prices lodging at each stop, catches towns that sell out and re-bases them, and produces an itinerary with a per-person and total cost table.

It reuses two sibling skills: `apify-plan-flights` for airfare and `apify-plan-hotel-stays` for lodging. Read those for the depth on each Actor; this skill is the orchestration and the trip judgment.

## When to use this skill

- The traveler says "plan a trip to", "help me plan a trip for this event", or "flights and hotels for these dates".
- The trip is anchored to a fixed date or place: a game, a festival, a wedding, a conference, an open house.
- They want an itinerary and a budget, not just one flight search or one hotel search.
- The plan involves flying in and driving, or more than one place to sleep.

Not for: a single hotel comparison (use `apify-plan-hotel-stays`) or a single flight search (use `apify-plan-flights`).

## Gather the trip parameters first

Ask for what is missing before you search. Everything downstream maps to these:

- Origin airport (or home city).
- Anchor: a fixed event (with its expected date) or a destination and rough dates.
- Party size and rooms.
- Budget range for the whole trip, or per night and per ticket.
- Preferences: airport vs city-center lodging, nonstop vs cheapest flights, one base vs splitting nights.
- Rewards programs: an airline or miles program for flights, a hotel program for lodging. Pass both to `scripts/rewards.py`.

## Workflow

1. Anchor first, and verify it live. The whole plan hinges on the fixed date being right. Event dates and venue details are present-day facts, so confirm them with a web search rather than trusting memory. Example: a Trinity Site open house is a specific Saturday each October; verify the exact date and the gate location before building around it.

2. Price the flights. Follow `apify-plan-flights`: resolve airports, run the search for the party and dates, read `price_insights` to judge the fare, and note the arrival and departure days. Those days set the first and last nights of lodging. Apply the airline or miles program via the `airlines` filter.

3. Work out the driving and the shape of each day. Look up or compute the drive legs between the arrival airport, the anchor, and each candidate base. Reason about whether a leg is a day trip or needs an overnight. Distance decides lodging shape: if the anchor is far from the arrival city, base near the anchor for that night; if it is close, a day trip from one base is cheaper and simpler. Example: Denver to the Albuquerque area is about 6.5 to 7.5 hours of driving, and the Trinity gate is about an hour from one base but nearly two from another, which is what makes the middle nights an overnight near the site rather than a daily round trip.

4. Price lodging at each overnight stop. Follow `apify-plan-hotel-stays` for each base and date: run the search, apply the hotel rewards brand filter, and normalize per-night versus total. Run the arrival-night and departure-night lodging near the airport, and the anchor nights near the anchor.

5. Detect sellouts and re-base automatically. Apply the availability guard: if a small town near the anchor comes back largely with no rates for the anchor night, it is selling out. Say so and widen to a larger nearby city, then re-price. Example: Socorro sold out for the open-house night, so the base moved to Albuquerque about an hour north.

6. Assemble the itinerary and the cost. Lay out the days (fly in, drive, anchor day, drive back, fly home), then total the cost: flights for the party, lodging per night summed across stops, and a note on rental cars or fuel if relevant. Show per-person and total. Label every lodging basis (per night for a room, or a full multi-night total with fees for a rental).

7. Render the deliverables. Use `scripts/render_price_table.py` for an email-ready HTML table and a Markdown version: a flights section, a lodging section per stop, and a cost summary, each with booking links and a "verify before booking" note.

## The Actors

- Flights: https://apify.com/johnvc/Google-Flights-Data-Scraper-Flight-and-Price-Search?fpr=9n7kx3&fp_sid=awesomeskills (Actor ID `johnvc/Google-Flights-Data-Scraper-Flight-and-Price-Search`)
- Hotels: https://apify.com/johnvc/google-hotels-search-scraper?fpr=9n7kx3&fp_sid=awesomeskills (Actor ID `johnvc/google-hotels-search-scraper`)
- Both are pay per event; estimate before big runs (see `references/gotchas.md`).

## Prerequisites

- Apify account (sign up at https://apify.com?fpr=9n7kx3&fp_sid=awesomeskills); `apify login` or an `APIFY_TOKEN`.
- Web search, to verify the anchor date and drive times.
- Optional: Chrome MCP, only to price specific Airbnb listings.

## Run it with the Apify CLI

Flights leg:

```bash
apify actors call "johnvc/Google-Flights-Data-Scraper-Flight-and-Price-Search" -i '{"departure_id":"IAD","arrival_id":"DEN","outbound_date":"2026-10-15","return_date":"2026-10-19","adults":3,"currency":"USD"}' \
  --json \
  --user-agent apify-awesome-skills/apify-plan-a-trip \
  2>/dev/null
```

Lodging leg (arrival night near the airport):

```bash
apify actors call "johnvc/google-hotels-search-scraper" -i '{"search_type":"search","q":"hotels near Denver International Airport","check_in_date":"2026-10-15","check_out_date":"2026-10-16","adults":3,"currency":"USD","max_pages":1}' \
  --json \
  --user-agent apify-awesome-skills/apify-plan-a-trip \
  2>/dev/null
```

Every call carries the three flags this repo expects: `--json`, `--user-agent apify-awesome-skills/apify-plan-a-trip`, and `2>/dev/null`.

## Run it from Claude or another AI agent (MCP)

Both Actors are MCP-ready:

`https://mcp.apify.com/?tools=actors,docs,johnvc/Google-Flights-Data-Scraper-Flight-and-Price-Search,johnvc/google-hotels-search-scraper`

Then ask, for example: "Plan a 4-night trip: fly IAD to DEN Oct 15 to 19 for 3 adults, we are driving to the Trinity Site open house on Oct 17, base us sensibly, and give me an itinerary and total cost."

## Cost guardrails

Each Actor is pay per event; a normal trip runs a handful of pages across flights and a few lodging searches, usually well under a dollar. `max_pages` is the cost cap on hotels; leave `fetch_booking_options` off on flights unless links are needed. See `references/gotchas.md` and the two sibling skills.

## Honest limits

- Read-only. This plans and prices; it does not book anything.
- Cash prices only, not points or award pricing.
- Drive times and event dates are looked up, not guaranteed; the traveler should verify before booking.
- Airbnb is priced separately from Google Hotels (see the hotels skill references).

## Troubleshooting and depth

- Flight depth, price insights, and the flights parse routine: the `apify-plan-flights` skill and its `references/gotchas.md`.
- Lodging depth, the availability guard, and the Airbnb and subagent-parse routines: the `apify-plan-hotel-stays` skill and its references.
- Trip judgment encoded as lessons: `references/trip-planning-lessons.md`.
- Actor routing table: `references/actor-index.md`.

## Bundled scripts

- `scripts/render_price_table.py`: rows in, a Markdown table and an email-ready HTML table out.
- `scripts/rewards.py`: a rewards program name in, airline codes or hotel brand families out.

## Related travel skills

- `apify-plan-flights`: airfare only.
- `apify-plan-hotel-stays`: lodging only.
- `apify-scrape-hotel-prices`, `apify-google-flights-api`: the raw-data (API and dataset) versions.
