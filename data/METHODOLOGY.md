# Methodology — SafeStreets US Walkability Dataset

This dataset exists because the raw inputs are public but **fragmented, messy, and not analysis-ready**. The value here is the cleaning, joining, derived signals, and LLM-ready packaging — every number below is traceable to an open source so you can audit or reproduce it.

## What this is
One row per **US census block group** (219,586 populated block groups, 938 metros, 50 states + DC — the full block-group national release; the earlier file covered 34,309 metro tracts). It combines a walkability measure, affordability, commercial activity, transit, and demographics into a single analysis-ready table, plus four derived flags and a curated list of "Commercial Deserts."

## The core walkability measure
`walkability_index` is the **EPA National Walkability Index** (Smart Location Database V3), a 1–20 score built from:
- street intersection density (network connectivity),
- proximity to transit,
- employment & household mix (destinations within reach).

We use EPA's published index directly and **do not modify it** — that keeps the core number verifiable. EPA's index is free; what you're paying for is everything joined around it.

## What the EPA index cannot see, and what we did about it

The EPA index is built entirely from street-network geometry and land-use mix. There is no crash data anywhere in it. It counts intersections; it never asks what happens to you at one.

That blind spot is measurable. The EPA scores the Las Vegas metro at 11.74 and the Philadelphia metro at 11.75 — statistically the same place. Las Vegas earns it on intersection density (132 per sq mi against Philadelphia's 101), because its suburbs are laid out on a fine grid. But those intersections are multi-lane arterial crossings. A pedestrian in the Las Vegas metro is killed at **3.72 per 100k per year against Philadelphia's 2.38** — 56% higher, on an index that says the two are identical.

So we added the layer the EPA is missing:

`safe_walk_score` = **0.7 × walkability_pctile + 0.3 × ped_safety_pctile**

Both terms are national percentile ranks, so the blend sits on a common 0–100 scale. (This is also how EPA builds its own index: rank the components, then weight them.) The 70/30 split matches the Street Safety weight already used by SafeStreets' live 15-minute-city score, so the dataset and the site agree with each other.

Under `safe_walk_score`, Las Vegas drops from level-with-Philadelphia to 14 points below it, and the national ranking stops producing results an urbanist would laugh at.

We also ship `walk_safety_gap` = `walkability_pctile − ped_safety_pctile`, which measures **how much a place flatters itself**. Miami (+52.9) and Las Vegas (+48.6) are built like you could walk and kill you if you try. Boston (−15.9) and Minneapolis (−21.4) are safer than their street layout suggests.

> **Limitation, stated plainly.** FARS crash data is county-level, so the safety term is coarser than the block-group walkability term. A quiet residential street inside a deadly county carries its county's risk. This is the best national join currently possible: FARS publishes no finer geography that supports exposure-normalised rates. Block-group-precise modeled crash risk is on the roadmap. Where FARS has no county record (mostly rural), `safe_walk_score` is blank rather than guessed — 185,028 of 219,586 block groups are scored.

> **Honest scope note.** `walkability_index` is EPA's, unmodified and free to download. `safe_walk_score`, `walk_safety_gap` and the enrichment layers are ours. This is also *not* SafeStreets' 4-component address score (Daily Reach / Street Safety / Transit Reach / Walking Comfort), which is computed live per address at safestreets.streetsandcommons.com — that one runs on a single address, this one runs on all 219,586 neighborhoods at once.

## The enrichment layers (what we added)
| Layer | Source | What we did |
|---|---|---|
| Income & home value | US Census ACS 2020–2024 (2024 5-yr release) | Joined by tract; **shipped as reported** — no synthetic inflation. (`zillow_home_value` carries the current market price.) |
| Affordability | derived | `value_to_income_ratio` and a national affordability percentile. |
| Commercial activity & gap | EPA SLD employment + establishment counts | Built `commercial_activity_index`, then `commercial_gap_pctile` = walkable/dense but commercially empty. |
| Transit | EPA SLD | Service-frequency and access percentiles. |
| Underserved demand | derived | Composite of density, affordability, and the commercial gap — the site-selection signal. |
| Commercial Deserts | derived + curated | The 81 tracts that are walkable, dense, affordable, yet business-starved. |

## Derived thresholds
- **Most walkable**: `walkability_index ≥ 15.26` (EPA's own "most walkable" cutoff).
- **Affordable**: `value_to_income_ratio ≤ 3.0` (standard housing-affordability convention).
- **Walkable + affordable**: both true (2,967 block groups, ~1.4% on 2020–2024 ACS).
- **Commercial Desert**: top of the underserved-demand distribution among walkable, dense, affordable tracts — manually reviewed to 81.

## Pedestrian safety layer (county-level, by road type)
Each tract carries its **county's** pedestrian-safety profile, and `pedestrian_safety_by_county.csv` holds the full breakdown for 953 counties.

- **Source:** NHTSA FARS, pedestrian fatalities 2022–2024 (21,915 with coordinates).
- **Geocoding:** each crash is assigned to a census block group by nearest centroid, then rolled up to its county (the first 5 digits of the GEOID).
- **Road type:** from the FARS functional class (`func_class`) and the surface-arterial flag. Nationally ~62% of pedestrian deaths are on arterials; the county file splits arterial / highway / collector / local.
- **Why county, not block group:** 21,915 deaths across 219,586 block groups over three years is far below one per block group — most would read "0," which means *small sample*, not *safe*. Counting at that level would publish noise that looks reassuring. County aggregates are stable; the road-type and after-dark shares are rates, so they compare cleanly across places. Tract-precise, modeled crash risk is the planned v2.

## Data sources (all public)
- **EPA Smart Location Database V3** — walkability index, density, employment, transit.
- **US Census ACS 2020–2024** (2024 5-year release) — income, home value (as reported). Population & households come from EPA SLD's ACS-based demographics.
- **Zillow ZHVI** — current market home value (Apr 2026), the market-price counterpart to the ACS survey value.
- **NHTSA FARS 2022–2024** — pedestrian fatalities, road type, time of day (the county safety layer).

## Known limitations (stated plainly)
- Tract resolution: a tract can blend a walkable core with a car-dependent edge.
- Cross-sectional: this is a single snapshot, not a time series. Treat "where a neighborhood is heading" as a hypothesis, not a measurement.
- Income/home value are ACS 2020–2024 5-year estimates **as reported** (not projected to the current year); `zillow_home_value` provides the current market price where you need it.
- `transit_service_frequency` is blank where EPA had no record; absence ≠ zero service.

## Reproducibility & contact
Questions, corrections, or a custom cut (specific metros, additional layers, block-group resolution): **safestreets.streetsandcommons.com** → Contact, or reply to your purchase receipt.
