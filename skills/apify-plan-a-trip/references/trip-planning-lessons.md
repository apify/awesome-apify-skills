# Trip-planning lessons (the judgment worth encoding)

These are the decisions that made a real event trip work (a fly-to-Denver, drive-to-New-Mexico trip anchored on a Trinity Site open house). They are the difference between a data dump and a plan. Read this when a trip involves an anchor event, driving, or multiple places to sleep.

## Anchoring and verification

1. Anchor on the fixed thing first. The plan hangs off the immovable date or place (the event, the wedding, the conference). Build outward from it: flights to arrive before it, lodging for the nights around it.
2. Verify the anchor live. Event dates and venue details are present-day facts and they change. Confirm the date and location with a web search before building the plan, and tell the traveler to reconfirm before booking.

## Driving and the shape of the trip

3. Drive time decides lodging shape. Compute or look up the legs between the arrival airport, the anchor, and each candidate base. A long leg turns a visit into an overnight; a short one keeps it a day trip from a single base. This is the call that sets where to sleep.
4. Base near the anchor for the anchor night when the drive is long, and near the airport for the arrival and departure nights. Splitting the trip into an airport base and an anchor base is often cheaper and less tiring than one central base with long daily drives.

## Lodging judgment

5. No rate means no availability, not a price of zero. Separate priced from unpriced properties when you flatten results.
6. Small towns sell out around events. When the well-rated properties in a small town near the anchor come back without rates for the anchor night, treat it as a sellout, say so, and widen the search to a larger nearby city, then re-price.
7. Normalize before comparing. Hotel rates are per room for the party; whole-home rental totals fold in cleaning and service fees, which make a one-night rental look expensive per night. Label the basis (per night for a room, or a full multi-night total with fees) so the comparison is fair.
8. Rank on rating, review count, and price, not on star class. Star class is occasionally wrong; show it as a label only.

## Cost and delivery

9. Total the trip, not just the pieces. Sum flights for the party, lodging per night across every stop, and note rental car or fuel where it matters. Show per-person and total, because that is the number the traveler actually decides on.
10. Deliver something shareable. A full comparison document (Markdown) and an email-ready HTML price table are what people forward and act on. Include booking links, label per-night versus total clearly, flag availability and cancellation windows, and add a "verify before booking" note. Use `scripts/render_price_table.py`.

## Data handling (so the plan stays accurate and the context stays clean)

11. Parse big result files in a subagent. Each search returns tens of KB of minified JSON in a tool-results file; hand the file paths to one subagent that returns compact, verified rows rather than pulling raw JSON into the main context.
12. Compute on the numeric fields, not the display strings, and dedupe hotels on `property_token`. See the two sibling skills' references for the exact field maps and the Airbnb routine.
