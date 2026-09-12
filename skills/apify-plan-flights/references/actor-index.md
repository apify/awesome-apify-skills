# Actor index: plan flights

The primary Actor for this skill, plus the travel-data Actors worth chaining. Read this after `SKILL.md` to pick the right Actor for a specific user intent.

| Platform | User intent | Actor ID | Tier | Notes |
|----------|-------------|----------|------|-------|
| Google Flights | Price and compare flights for a route and dates | `johnvc/Google-Flights-Data-Scraper-Flight-and-Price-Search` | community | Pay per page plus a per-run setup fee. Supports one-way, round-trip, and multi-city (via `multi_city_json`). Returns `results.best_flights`, `results.other_flights`, and `price_insights`. Booking links only when `fetch_booking_options` is true (billed per option). |

## Chain with other travel-data Actors

| User intent | Actor ID | Notes |
|-------------|----------|-------|
| Lodging for the same trip | `johnvc/google-hotels-search-scraper` | Pair airfare with hotels for a total trip cost. See the `apify-plan-hotel-stays` skill. |
| A whole event or destination trip with logistics | both Actors | See the `apify-plan-a-trip` skill; it reuses this flight workflow. |
| Destination discovery and trip ideas | `johnvc/google-travel-explore-api` | Find where to fly before pricing a route. |

## How to extend

1. Search candidates: `apify actors search "google flights" --json --limit 20 2>/dev/null`
2. Fetch the input schema: `apify actors info "johnvc/Google-Flights-Data-Scraper-Flight-and-Price-Search" --input --json 2>/dev/null`
3. Add a row above with the user intent that should trigger it.

Note: `Tier` here is `community` because these are third-party Actors published by John Cole on the Apify Store, not Apify-maintained Actors.
