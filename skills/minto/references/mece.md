# MECE Groupings

Every grouping in the pyramid must be **Mutually Exclusive and Collectively Exhaustive**.

---

## Mutually Exclusive

No item in the group overlaps with any other. Each idea occupies its own territory.

**Test:** take any two items in the group. Ask: "could a piece of evidence support both?" If yes, the items overlap. Redraw the boundary.

Wrong (overlapping):
- Improve customer satisfaction
- Reduce support tickets
- Improve user experience

"Reduce support tickets" is a subset of both others. These are not exclusive.

Right (exclusive):
- Reduce time-to-resolution for support tickets
- Increase self-service completion rate
- Improve first-contact resolution rate

Each measures a different dimension. No overlap.

## Collectively Exhaustive

The group covers all relevant territory. Nothing important is missing.

**Test:** ask "if all of these are true, does the key-line point necessarily follow?" If someone could object with "yes, but what about X?" then X is missing.

Wrong (gap):
- Revenue increased 20%
- Customer count grew 15%

Missing: margin, retention, unit economics. The group does not prove "the business is healthy."

Right (exhaustive):
- Revenue increased 20% with stable margins
- Customer count grew 15% with 95% retention
- Unit economics turned positive in Q2

No reasonable objection remains.

## The Same-Kind Test

Every item in a MECE group must be the same type. Find a plural noun that labels all items.

| Plural noun | Items must be... |
|-------------|-----------------|
| Reasons | ...why something is true |
| Steps | ...in a sequential process |
| Problems | ...that need solving |
| Recommendations | ...actions to take |
| Requirements | ...that must be met |
| Risks | ...that could occur |

If you cannot find one noun, the grouping mixes types. Split into separate groups.

## Optimal Group Size

- **3 items** — optimal. Easy to hold in memory. Strong rhetorical effect.
- **4-5 items** — acceptable. Reader can still track.
- **6+ items** — too many. Sub-group into categories of 3.
- **2 items** — suspicious. Often means a third is hiding, or the two items belong at different levels.
- **1 item** — not a group. Promote it to the level above or find its siblings.
