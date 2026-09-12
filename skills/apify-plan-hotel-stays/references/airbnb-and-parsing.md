# Parsing large results, and pricing Airbnb directly

Two routines the workflow points to: how to parse big hotel result files without flooding the main context, and how to price specific Airbnb listings that Google Hotels never returns.

## Parse hotel results in a subagent

Why: one search returns 50 to 85 KB of minified, single-line JSON. It exceeds the inline tool-result cap, so it is written to a tool-results file (often under a path like `/var/folders/...`). Those files open with the `Read` tool but not from a bash sandbox, and line-based chunking does not help because the JSON is one line. Pulling several of these into the main context burns hundreds of KB fast.

The efficient pattern:

1. Run all the searches you need first. Collect the saved file path from each run.
2. Hand the paths to one subagent with a tight instruction: read each file, pull the `properties` array, and return a compact table with one row per property.
3. Ask the subagent for exactly these fields: `name`, `type`, `extracted_hotel_class`, `overall_rating`, `reviews`, `rate_per_night.extracted_lowest`, `total_rate.extracted_lowest`, `gps_coordinates`, `link`, `property_token`. Sorted by price ascending.
4. Tell the subagent to verify the numbers it reports against the file, not to estimate, and to mark any property with no `rate_per_night` as "no rate returned" rather than dropping it silently.

Field projection note: requesting `fields="properties.name"` returns empty objects, because `properties` is an array, not a nested object. Request `fields="properties"` (the whole array) or the whole item. `omit` does not trim nested array subfields either, so there is no server-side way to slim these; trim client-side in the subagent.

The subagent returns a small ranked table. Feed those rows to `scripts/render_price_table.py`.

## Price a specific Airbnb listing (Chrome MCP)

Why: the Actor's `vacation_rentals: true` returns Vrbo, Whimstay, and property-manager sites, never airbnb.com. To price a real Airbnb listing you scrape airbnb.com directly.

Routine:

1. Strip tracking parameters from any user-supplied Airbnb URL. Keep only the room id.
2. Navigate to `https://www.airbnb.com/rooms/ROOM_ID?check_in=YYYY-MM-DD&check_out=YYYY-MM-DD&adults=N` with the Chrome MCP tools.
3. Read the page text and extract: title, location, beds and baths, guest capacity, rating, review count, the all-in total for the dates, cancellation policy, and house rules.
4. Record the total as a "2-night total, all fees included" style figure and compute a per-night figure for context. Label the basis so it compares fairly against per-room hotel rates.

Add each Airbnb row to the same row set you pass to `render_price_table.py`, in its own section, with the booking URL as the link.
