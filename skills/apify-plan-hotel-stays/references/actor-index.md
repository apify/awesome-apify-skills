# Actor index: plan hotel stays

The primary Actor for this skill, plus the travel-data Actors worth chaining when a task needs more than hotel listings. Read this after `SKILL.md` to pick the right Actor for a specific user intent.

| Platform | User intent | Actor ID | Tier | Notes |
|----------|-------------|----------|------|-------|
| Google Hotels | Price and compare hotels and rentals for a destination and stay date | `johnvc/google-hotels-search-scraper` | community | Pay per page (about 20 properties per page) plus a per-run setup fee. Modes via `search_type`: `search` (default), `autocomplete`, `photos`, `reviews`. A query search needs `q` plus `check_in_date`; photos and reviews need a `property_token`. One page object per dataset item. |

## Chain with other travel-data Actors

| User intent | Actor ID | Notes |
|-------------|----------|-------|
| Flight prices and route data for the same trip | `johnvc/Google-Flights-Data-Scraper-Flight-and-Price-Search` | Pair lodging with airfare for a total trip cost. See the `apify-plan-flights` skill. |
| A whole event or destination trip with logistics | both Actors | See the `apify-plan-a-trip` skill; it reuses this lodging workflow. |
| Destination discovery and trip ideas | `johnvc/google-travel-explore-api` | Find where to search before pricing hotels. |
| Restaurants and attractions near a property | `johnvc/google-local-api` | Feed a property's `gps_coordinates` area into local search. |

## How to extend

1. Search candidates: `apify actors search "google hotels" --json --limit 20 2>/dev/null`
2. Fetch the input schema: `apify actors info "johnvc/google-hotels-search-scraper" --input --json 2>/dev/null`
3. Add a row above with the user intent that should trigger it.

Note: `Tier` here is `community` because these are third-party Actors published by John Cole on the Apify Store, not Apify-maintained Actors.
