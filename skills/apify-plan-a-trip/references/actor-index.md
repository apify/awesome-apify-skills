# Actor index: plan a trip

This skill orchestrates two Actors plus web search. Read this after `SKILL.md` to route a specific need to the right Actor.

| Platform | User intent | Actor ID | Tier | Notes |
|----------|-------------|----------|------|-------|
| Google Flights | Price flights for each leg of the trip | `johnvc/Google-Flights-Data-Scraper-Flight-and-Price-Search` | community | One-way, round-trip, multi-city. Returns `results.best_flights`, `results.other_flights`, `price_insights`. See the `apify-plan-flights` skill for depth. |
| Google Hotels | Price lodging at each overnight stop | `johnvc/google-hotels-search-scraper` | community | Pay per page; one page object per dataset item. Apply the availability guard and rewards brand filter. See the `apify-plan-hotel-stays` skill for depth. |
| Web search | Verify the anchor event date and look up drive times | (built-in) | n/a | Event dates and distances are present-day facts; verify them live, do not assume. |

## Chain with other travel-data Actors

| User intent | Actor ID | Notes |
|-------------|----------|-------|
| Destination discovery before dates are fixed | `johnvc/google-travel-explore-api` | Find where to go, then anchor and plan. |
| Restaurants and attractions at a stop | `johnvc/google-local-api` | Feed a base city or a property's coordinates into local search. |

## How to extend

1. Search candidates: `apify actors search "travel" --json --limit 20 2>/dev/null`
2. Fetch a schema: `apify actors info "johnvc/google-hotels-search-scraper" --input --json 2>/dev/null`
3. Add a row above with the user intent that should trigger it.

Note: `Tier` here is `community` because these are third-party Actors published by John Cole on the Apify Store, not Apify-maintained Actors.
