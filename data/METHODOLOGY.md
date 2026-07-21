# Methodology: Walkable America (national sample)

This dataset exists because the raw inputs are public but fragmented, messy, and not analysis-ready. The value is the cleaning, joining, derived signals, and LLM-ready packaging. Every number below is traceable to an open source so you can audit or reproduce it.

## What this is

One row per Census block group. This is a national sample: 6,000 block groups across 716 metros and 51 states and territories. It combines a walkability measure, affordability, commercial activity, transit, and demographics into a single analysis-ready table, plus derived flags and a county pedestrian-safety profile. Percentile columns are ranked against all 219,586 US block groups, so they read as national position.

## The core walkability measure

`walkability_index` is the EPA National Walkability Index (Smart Location Database v3), a 1 to 20 score built from:
- street intersection density (network connectivity),
- proximity to transit,
- employment and household mix (destinations within reach).

We use EPA's published index directly and do not modify it, which keeps the core number verifiable. EPA's index is free. The value here is everything joined around it.

## What the EPA index cannot see, and what we did about it

The EPA index is built entirely from street-network geometry and land-use mix. There is no crash data anywhere in it. It counts intersections; it never asks what happens to you at one.

The blind spot is measurable. The EPA scores the Las Vegas metro at 11.74 and Philadelphia at 11.75, statistically the same place. Las Vegas earns it on intersection density (132 per sq mi against Philadelphia's 101), because its suburbs are laid out on a fine grid. But those intersections are multi-lane arterial crossings. A pedestrian in the Las Vegas metro is killed at 3.72 per 100k per year against Philadelphia's 2.38, 56% higher, on an index that calls the two identical.

So we added the layer the EPA is missing:

`safe_walk_score` = 0.7 x `walkability_pctile` + 0.3 x `ped_safety_pctile`

Both terms are national percentile ranks, so the blend sits on a common 0-100 scale. This is also how EPA builds its own index: rank the components, then weight them. The 70/30 split matches the Street Safety weight already used by SafeStreets' live 15-minute-city score. Under `safe_walk_score`, Las Vegas drops from level-with-Philadelphia to 14 points below it.

We also ship `walk_safety_gap` = `walkability_pctile` - `ped_safety_pctile`, which measures how much a place flatters itself. Miami (+52.9) and Las Vegas (+48.6) are built like you could walk and kill you if you try. Boston (-15.9) and Minneapolis (-21.4) are safer than their street layout suggests.

Limitation, stated plainly: FARS crash data is county-level, so the safety term is coarser than the block-group walkability term. A quiet residential street inside a deadly county carries its county's risk. This is the best national join currently possible; FARS publishes no finer geography that supports exposure-normalised rates. Where FARS has no county record (mostly rural), `safe_walk_score` is left blank rather than guessed. 84% of rows are scored.

> **Scope note.** This is block group-level. It is not SafeStreets' own four-component address score (Daily Reach, Street Safety, Transit Reach, Walking Comfort), which is computed live per address at safestreets.streetsandcommons.com. This file is the screening layer. The live engine is the per-address layer.

## The enrichment layers (what we added)

| Layer | Source | What we did |
|---|---|---|
| Income and home value | US Census ACS 2020 to 2024 (2024 5-yr release) | Joined by block group, shipped as reported, no synthetic inflation. |
| Affordability | derived | `value_to_income_ratio` and a national affordability percentile. |
| Commercial activity and gap | EPA SLD employment and establishment counts | Built `commercial_activity_index`, then `commercial_gap_pctile` = walkable and dense but commercially empty. |
| Transit | EPA SLD | Service-frequency and access percentiles. |
| Underserved demand | derived | Composite of density, walkability, affordability, and the commercial gap. The site-selection signal. |
| Commercial Deserts | derived and curated | Block groups that are walkable, dense, affordable, yet business-starved. |

## Derived thresholds

- **Most walkable:** `walkability_index` at or above 15.26 (EPA's own "most walkable" cutoff).
- **Affordable:** `value_to_income_ratio` at or below 3.0 (Demographia median-multiple convention).
- **Walkable and affordable:** both true. About 1% of US households nationally, 80 block groups in this sample.
- **Commercial Desert:** top of the underserved-demand distribution among walkable, dense, affordable block groups, then manually reviewed.

## Pedestrian safety layer (county-level, by road type)

Each block group carries its county's pedestrian-safety profile.

- **Source:** NHTSA FARS, pedestrian fatalities 2022 to 2024.
- **Geocoding:** each crash is assigned to a Census block group by nearest block group centroid, then rolled up to its county (the first 5 digits of the GEOID).
- **Road type:** from the FARS functional class and the surface-arterial flag. Nationally about 62% of pedestrian deaths are on arterials.
- **Why county, not block group:** roughly 21,915 deaths across 219,586 block groups over three years is about 0.6 per block group. Most block groups would read "0," which means small sample, not safe. Counting at the block group level would publish noise that looks reassuring. County aggregates are stable, and the road-type and after-dark shares are rates, so they compare cleanly across places. Block group-precise, modeled crash risk is a separate build.

## Data sources (all public)

- **EPA Smart Location Database v3:** walkability index, density, employment, transit.
- **US Census ACS 2020 to 2024 (2024 5-year release):** income, home value (as reported). Population and households come from EPA SLD's ACS-based demographics.
- **NHTSA FARS 2022 to 2024:** pedestrian fatalities, road type, time of day (the county safety layer).
- **OpenStreetMap:** street network and walkable destinations underpinning the walkability context.

The full dataset adds Zillow ZHVI home values and ZORI rents (the market-price and renter-affordability columns), which are not in this free sample.

## Known limitations (stated plainly)

- This is a 6,000-block group national sample, not the full 219,586-block group set.
- Block group resolution: a block group can blend a walkable core with a car-dependent edge.
- Cross-sectional: this is a single snapshot, not a time series. Treat "where a neighborhood is heading" as a hypothesis, not a measurement.
- Income and home value are ACS 2020 to 2024 5-year estimates as reported, not projected to the current year.
- `transit_service_frequency` is blank where EPA had no record. Absence is not zero.

## Reproducibility and contact

Open methodology with a citable DOI: https://doi.org/10.5281/zenodo.20506270
Questions, corrections, or a custom cut (specific metros, additional layers, live API): safestreets.streetsandcommons.com.
