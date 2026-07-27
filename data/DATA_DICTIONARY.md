# Data Dictionary — SafeStreets US Walkability Dataset

One row = one **US census block group** (the finest Census neighborhood unit, ~600–3,000 people).
File: `us_walkability_full.csv` — **219,586 populated block groups** across 938 metro areas and every US state + DC. This is the full block-group national release (the tract-level file covered 34,309 metro tracts; this is ~6× the coverage, at finer resolution).

> Percentiles are **national, 0–100** (100 = highest in the country for that measure) unless noted.

> **Attribution.** Market-price and rent signals are **derivative works computed from Zillow Group data (ZHVI and ZORI)**. Raw Zillow index values are not redistributed; only SafeStreets-derived percentiles, ratios, and flags are shipped. Data provided by Zillow Group. Walkability, density, transit, and employment are from the EPA Smart Location Database; income and home value from US Census ACS; pedestrian safety from NHTSA FARS.

> **Getting live market values (optional).** This file ships derived market and rent signals, not raw Zillow dollar values. If you need the current market home value or rent in dollars, the `zillow_zcta` column gives the ZIP each neighborhood maps to; join it to Zillow's free public research data (ZHVI for home value, ZORI for rent, at zillow.com/research/data) on that ZIP to attach live figures. Or ask us for a custom cut with market values merged for your target geographies. The ACS `median_home_value` and `median_household_income` dollar figures are already in this file.

---

## How to read provenance (start here)

Every column below is tagged so you know exactly what you're holding. Perplexity, ChatGPT, and a careful human all ask the same first question — *which numbers are real measurements and which did you compute?* This answers it per column.

**Raw / Derived flag (`R/D`):**
- **R** — Raw pass-through from a public source (may be unit-converted only). Income and home value are shipped **as reported by the Census — not projected or inflated.**
- **D** — Derived: computed by SafeStreets from raw inputs. Formula in the [Derived-field formulas](#derived-field-formulas) section.
- **D-pct** — A national **percentile rank** (0–100) of a base column.
- **D-flag** — A boolean from a published threshold.
- **ID** — Identifier / join key (raw Census geography).

**Of the 48 columns: 4 are identifiers, 15 are raw measurements, 28 are derived, and 1 (`income_source`) flags how each block group's income was matched.** The derived fields are the product — the join, the normalization, and the decision shortcuts. Nothing here is a black box; every derived value traces to a raw input and a stated formula.

> **Vintage note (block-group release):** EPA's National Walkability Index is published on **2010-vintage block groups**. Census ACS income/home value is published on **2020-vintage block groups**. Where a block group's boundary is unchanged (≈78%), income joins exactly; where it was redrawn, income is transferred from the **nearest current block group** by population-weighted centroid. The `income_source` column records `exact` vs `nearest` per row so you always know which. Walkability, density, transit, and employment come straight from EPA with no crosswalk.

**Refresh cadence** is summarized in [Refresh schedule](#refresh-schedule) and tagged per column under *Vintage*.

---

## Master column table

| Column | Type | R/D | Source · Vintage | Description |
|---|---|---|---|---|
| `geoid` | string | ID | Census / EPA SLD v3 | **12-digit Census block group GEOID** (state+county+tract+block group). First 11 digits = tract, first 5 = county FIPS. Leading zeros preserved — read as text. Join key to any Census/ACS data. |
| `tract_name` | string | ID | Census / SafeStreets | Human-readable neighborhood name (parent-tract name where known, else metro/state). Column name kept as `tract_name` for schema continuity. |
| `metro` | string | ID | Census CBSA | Metro area the block group belongs to (blank for non-metropolitan block groups). |
| `state` | string | ID | Census | 2-letter state abbreviation. |
| `walkability_index` | float | **R** | EPA SLD v3 (2021, 2010 block groups) | **EPA National Walkability Index**, 1–20. Higher = more walkable street network + destination access. Used unmodified, native to the block group (no aggregation). |
| `walkability_pctile` | float | D-pct | derived from EPA SLD v3 | National percentile of `walkability_index`. |
| `county_ped_death_rate_per_100k` | float | **D** | NHTSA FARS 2022–2024 + block-group population | Annual pedestrian deaths per 100,000 residents in the parent county. `county_ped_deaths_3yr ÷ 3 ÷ county population × 100,000`. Blank where FARS has no county record. |
| `ped_safety_pctile` | float | D-pct | derived (FARS) | National percentile of pedestrian safety. **100 = safest** (lowest death rate). Inverted so that higher is always better, like every other percentile here. |
| `safe_walk_score` | float | **D** | **SafeStreets composite** | **The SafeStreets Walk Safety Score, 0–100.** `0.7 × walkability_pctile + 0.3 × ped_safety_pctile`. Answers the question the EPA index cannot: is this place laid out for walking *and* will walking there get you killed. The 70/30 weighting matches the Street Safety weight in SafeStreets' live 15-minute-city score. |
| `safe_walk_tier` | string | **D** | SafeStreets composite | Label for `safe_walk_score`: Walkable and safe (≥80) · Above average (≥60) · Average (≥40) · Below average (≥20) · Car-dependent or dangerous (<20). |
| `walk_safety_gap` | float | **D** | **SafeStreets composite** | `walkability_pctile − ped_safety_pctile`. **How much a place flatters itself.** Positive = built like you could walk, dangerous if you try (Las Vegas +48.6, Miami +52.9 at metro scale). Negative = safer than its street layout suggests (Boston −15.9, Minneapolis −21.4). This column exists in no other dataset. |
| `pop_density_per_sqmi` | float | **R** | EPA SLD v3 / Census | Population density, persons/sq mi (approx.). |
| `density_pctile` | float | D-pct | derived | National percentile of density. |
| `median_household_income` | int | **R** | ACS 2020–2024 (5-yr) | Median household income, USD. Census ACS 2024 5-year release, **as reported** (not inflation-adjusted). `exact` where the block-group boundary matches, else transferred from the nearest current block group (see `income_source`). Blank (~8%) where ACS suppressed the estimate. |
| `income_pctile` | float | D-pct | derived | National percentile of income. |
| `median_home_value` | int | **R** | ACS 2020–2024 (5-yr) | Median home value, USD — **ACS owner-reported**, as reported. Block-group level (finer than ZIP). Matched like income (see `income_source`). Runs ~15–20% below market because ACS is self-reported and topcoded. See `market_affordability_pctile` for a current-market affordability signal derived from Zillow. |
| `value_to_income_ratio` | float | **D** | derived (ACS) | `median_home_value ÷ median_household_income`. ≤3 = affordable, ≥5 = severely unaffordable (Demographia median multiple). |
| `affordability_pctile` | float | D-pct | derived | National percentile — **higher = more affordable** (lower value-to-income ratio). |
| `commercial_activity_index` | float | **D** | composite, from EPA SLD employment | Relative density of commercial/business activity, 0–1 scaled. Composite — see formulas. |
| `commercial_pctile` | float | D-pct | derived | National percentile of commercial activity. |
| `commercial_gap_pctile` | float | **D** | composite | **Higher = more underserved** — walkable/dense but commercially empty. The Commercial Deserts signal. |
| `transit_service_frequency` | float | **R** | EPA SLD v3 (D4c) | EPA aggregate transit service frequency per sq mi. **Blank where EPA had no record — blank ≠ zero.** |
| `transit_access_pctile` | float | D-pct | derived | National percentile of transit access. |
| `retail_emp_density` | float | **R** | EPA SLD v3 (D1C5_RET) | Retail employment density. |
| `service_emp_density` | float | **R** | EPA SLD v3 (D1C5_SVC) | Service employment density. |
| `entertainment_emp_density` | float | **R** | EPA SLD v3 (D1C5_ENT) | Entertainment employment density. |
| `retail_jobs` | int | **R** | EPA SLD v3 (E5_Ret) | Retail-sector jobs in the tract. |
| `service_jobs` | int | **R** | EPA SLD v3 (E5_Svc) | Service-sector jobs. |
| `entertainment_jobs` | int | **R** | EPA SLD v3 (E5_Ent) | Entertainment-sector jobs. |
| `population` | int | **R** | EPA SLD v3 (ACS base) | Total population (EPA SLD `TotPop`; SLD's demographics are ACS-derived). |
| `households` | int | **R** | EPA SLD v3 (ACS base) | Total households (EPA SLD `HH`). |
| `demand_income_pctile` | float | **D** | derived | Spending-power component of latent retail demand = `income_pctile`. |
| `demand_afford_pctile` | float | **D** | derived | Affordability component of latent retail demand = `affordability_pctile`. |
| `underserved_demand_score` | float | **D** | composite (transparent) | 0–100: unmet demand for daily-needs businesses in a walkable/dense/affordable-but-empty tract. The site-selection signal. Exact formula below. |
| `walkability_tier` | string | D-flag | derived (EPA cutoffs) | `Most walkable` (≥15.26), `Above average` (10.51–15.25), `Below average` (5.76–10.50), `Least walkable` (<5.76). |
| `is_most_walkable` | bool | D-flag | derived | `walkability_index ≥ 15.26` (EPA "most walkable" tier). |
| `is_affordable` | bool | D-flag | derived | `0 < value_to_income_ratio ≤ 3.0`. |
| `is_walkable_affordable` | bool | D-flag | derived | Both walkable and affordable. Rare — **2,967 block groups** (~1.4%). |
| `is_commercial_desert` | bool | D-flag | curated | Block group whose parent tract is one of the **81 curated Commercial Deserts** — walkable, dense, affordable, yet almost no businesses. See `commercial_deserts_81.csv`. |
| `county_ped_deaths_3yr` | int | **R** | NHTSA FARS 2022–2024 | Pedestrian fatalities in this tract's **county**, 2022–2024. County-level, not tract-precise (crashes too sparse to count per tract honestly — see methodology). |
| `county_pct_deaths_on_arterials` | int | **D** | rate, from FARS 2022–2024 | Share of those county deaths on surface arterials, %. The single strongest danger pattern. |
| `county_pct_deaths_after_dark` | int | **D** | rate, from FARS 2022–2024 | Share of those county deaths between 6pm–6am, %. |
| `zillow_zcta` | string | **D** | HUD/Census crosswalk | The 5-digit ZCTA/ZIP the market-price and rent signals were derived from (provenance). |
| `market_affordability_pctile` | float | D-pct | derived from Zillow ZHVI | National percentile of market-price affordability (100 = most affordable current market price relative to local income). A rank derived from Zillow ZHVI; the raw index value is not shipped. Coverage ~72%. |
| `price_to_rent_ratio` | float | **D** | derived (Zillow) | home value divided by annual rent (both derived from Zillow ZHVI and ZORI; raw index values are not shipped). The classic buy-vs-rent metric: **< 15 favors buying, 15 to 21 balanced, > 21 favors renting**. Refreshes monthly with the underlying Zillow layers. Nationally: 32% of covered neighborhoods favor buying, 37% balanced, 31% favor renting. |
| `rent_affordability_pctile` | float | **D** | derived (Zillow/ACS) | National percentile of renter affordability (100 = most affordable rent relative to local income, derived from Zillow ZORI rent relative to local income; the raw rent value is not shipped). Renter parallel to `affordability_pctile`. |
| `is_rent_burdened` | int (0/1) | **D** | derived (HUD threshold) | 1 where rent exceeds 30% of household income (HUD cost-burden line), else 0; derived from Zillow ZORI, blank where rent or income is missing. 40% of covered neighborhoods are rent-burdened; the share is **36% in the most-walkable quartile vs 6% in the least-walkable** (walkability carries a renter premium too). |
| `income_source` | string | **D** | provenance | How this block group's ACS income/home value was matched: `exact` (boundary unchanged 2010→2020, ~72%), `nearest` (boundary redrawn; value from the nearest current block group by population-weighted centroid, ~20%), or blank (income suppressed, ~8%). Lets you filter to exact-match rows if you want maximum precision. |

---

## Derived-field formulas

Exact formula for every derived column. As of the 2020–2024 ACS rebuild, **the entire income → affordability → demand chain is reproducible from this file** — no hidden weighting.

**Simple arithmetic (fully reproducible from this file):**
- `value_to_income_ratio` = `median_home_value / median_household_income`
- `walkability_tier` = step function on `walkability_index`: ≥15.26 → *Most walkable*; ≥10.51 → *Above average*; ≥5.76 → *Below average*; else *Least walkable* (EPA SLD cutoffs)
- `is_most_walkable` = `walkability_index ≥ 15.26`
- `is_affordable` = `0 < value_to_income_ratio ≤ 3.0`
- `is_walkable_affordable` = `is_most_walkable AND is_affordable`
- `is_commercial_desert` = the block group's parent tract ∈ the 81 manually-reviewed Commercial Deserts (`commercial_deserts_81.csv`)
- `county_pct_deaths_on_arterials` = arterial pedestrian deaths ÷ all pedestrian deaths in the county (FARS 2022–2024), ×100
- `county_pct_deaths_after_dark` = pedestrian deaths 6pm–6am ÷ all pedestrian deaths in the county, ×100

**Percentiles (`*_pctile`)** = national rank of the base column across all 219,586 block groups, scaled 0–100 (average-tie rank ÷ N × 100). `affordability_pctile` is **inverted** (100 = lowest value-to-income ratio = most affordable); all others run in the natural direction (100 = highest).

**The demand model (fully transparent):**
- `demand_income_pctile` = `income_pctile` — spending-power proxy (higher-income tracts demand more/higher-value retail).
- `demand_afford_pctile` = `affordability_pctile` — affordability proxy (residents with disposable income to spend locally).
- `underserved_demand_score` = `clip( 0.6 × demand + 0.4 × (100 − commercial_pctile), 0, 100 )`, where
  `demand = 0.35 × density_pctile + 0.25 × walkability_pctile + 0.20 × demand_income_pctile + 0.20 × demand_afford_pctile`.
  High = a tract with the residents and foot-traffic to support daily-needs retail (dense, walkable, with spending power) but **low commercial supply**. The site-selection signal. Pairs with `commercial_gap_pctile`.

**Composites (upstream EPA enrichment — inputs and direction exact):**
- `commercial_activity_index` — relative density of commercial/business activity, built from EPA SLD employment + establishment counts, 0–1 scaled. Higher = more active commercial fabric.
- `commercial_gap_pctile` — the **gap** between walkability/density and `commercial_activity_index`. Higher = walkable and dense but commercially empty.

---

## Refresh schedule

What updates, how often, and when to expect the next vintage.

| Source | Columns it feeds | Cadence | Next refresh |
|---|---|---|---|
| **Zillow ZHVI** (market price, derived) | `market_affordability_pctile`, `price_to_rent_ratio` | **Monthly** | Derived signals regenerated monthly from Zillow ZHVI (raw values not shipped). |
| **Zillow ZORI** (rent, derived) | `rent_affordability_pctile`, `is_rent_burdened`, `price_to_rent_ratio` | **Monthly** | Derived signals regenerated monthly from Zillow ZORI (raw values not shipped). |
| **US Census ACS 5-yr** | `median_household_income`, `median_home_value` (+ the ratios/percentiles/demand built on them) | Annual (Dec) | Current base = **2020–2024** (latest 5-yr release). 2021–2025 ≈ Dec 2026. |
| **NHTSA FARS** | `county_ped_deaths_3yr`, `county_pct_deaths_on_arterials`, `county_pct_deaths_after_dark` | Annual | 2023–2025 file ≈ late 2026. |
| **EPA Smart Location Database v3** | `walkability_index`, density, transit, jobs/employment, population, households | Irregular (EPA; v2 2014 → v3 2021) | Whenever EPA publishes v4 — no fixed date. |
| **Derived** (percentiles, flags, demand) | all `D` / `D-pct` / `D-flag` columns | Recomputed on every rebuild | — |

The fastest-moving inputs are Zillow (monthly) and ACS (annual); EPA walkability is the slow-moving backbone. A monthly regenerate keeps the market-price column current without touching the stable survey layer.

---

## The price columns — three angles on cost, on purpose

Cost is covered from three directions so you can answer both *buy* and *rent* questions, and cross-check survey against market. This is deliberate, not duplication:

| Column | What it is | Source · Vintage | Resolution | Best for |
|---|---|---|---|---|
| `median_home_value` | Owner-reported home value | ACS 2020–2024 (as reported) | **Tract** (finer) | Within-metro comparison; the survey baseline |

- **Own vs rent:** `value_to_income_ratio` (≤3 affordable, Demographia) answers ownership; `rent_affordability_pctile` and `is_rent_burdened` (HUD 30% line) answer renting. A neighborhood can be punishing to buy but reasonable to rent, or the reverse, and both sides are covered.
- **Buy vs rent:** `price_to_rent_ratio` (below 15 favors buying, above 21 favors renting) is the buy-versus-rent signal, and `market_affordability_pctile` ranks current-market affordability. Both are Zillow-derived; raw Zillow values are not shipped.
- `zillow_zcta` tells you exactly which ZIP both Zillow figures came from.

---

## The 15-Minute City framework — and what's actually in this file

Be precise about this, because it's the most common misread.

**What `walkability_index` in this file IS:** the **EPA National Walkability Index** (Smart Location Database v3) — a static, national, tract-level score from three measured inputs: street-intersection density (network connectivity), proximity to transit, and employment/household mix (destinations within reach). It is the best *static national* proxy for "can you get around here on foot," and we use it **unmodified** so the core number stays independently verifiable.

**What it is NOT:** it is not SafeStreets' own live 15-minute-city score. That score is a different, finer engine and is **not a column in this dataset**.

### The SafeStreets 15-Minute City score (the live engine, for context)
At `safestreets.streetsandcommons.com`, every **address** is scored live, 0–10, from four components (this is our methodology, not the EPA's):

| Component | Weight | What it measures |
|---|---|---|
| **Daily Reach** | 40% | How many of 7 daily-needs categories (grocery, healthcare, education, recreation, dining, shopping, civic) are reachable on foot. |
| **Street Safety** | 30% | Weighted-OR of crossing density and pedestrian separation (car-free / separated paths). A stroad sinks; a busy grid or a car-free core both score high. |
| **Transit Reach** | 15% | Frequency-weighted transit access (GTFS headways, not just stop presence). |
| **Walking Comfort** | 15% | Shade, tree canopy, heat stress, and the pedestrian environment. |

Tiers (0–10 display): **Pedestrian-first** ≥9.3 · **Very walkable** ≥8.3 · **Walkable** ≥7.0 · **Moderate** ≥5.5 · **Car-dependent** ≥3.5 · **Hostile** <3.5.

**Why two different measures?** Running a live, multi-API, per-address computation across all 219,586 block groups isn't practical to ship as a static file, and the EPA index is the most defensible national constant. The live engine is the per-address layer; this dataset is the national screening layer. **This file is that block-group-resolution release (~220,000 neighborhoods)** — the finest Census unit, national. Need live per-address SafeStreets scores at volume? That's the API — ask.

> The 15-minute-city *target* (daily needs within a 15-min walk/bike) is Carlos Moreno's (Sorbonne, 2016). See `FRAMEWORKS_AND_STANDARDS.md` §1 for the cited distance/speed thresholds.

---

## Curated list files (bonus)
Pre-filtered answers, drawn from the same engine. All `geoid`/percentile/threshold conventions match the main file.

**`walkable_affordable_neighborhoods.csv`** — the 2,967 block groups that are both walkable (`walkability_index` ≥ 15.26) and affordable (`value_to_income_ratio` ≤ 3.0). On 2020–2024 ACS this is rarer than older vintages implied (~1% of households) — walkable urban cores are genuinely expensive.
| Column | Type | Description |
|---|---|---|
| `geoid` | string | 12-digit block group GEOID (text, leading zeros). |
| `tract_name`, `metro`, `state` | string | Identity. |
| `walkability_index` | float | EPA index, 1–20. |
| `median_home_value`, `median_household_income` | int | USD (ACS 2020–2024, as reported). |
| `value_to_income_ratio` | float | ≤ 3.0 by construction (ACS-based). |
| `population` | int | ACS population. |

**`walkable_affordable_metros.csv`** — the above rolled up by metro: `metro`, `neighborhood_count`, `population`, `median_home_value`, `median_walkability_index`, `zillow_median_home_value` (population-weighted Zillow ZHVI across the metro's walkable+affordable tracts).

**`car_free_spots.csv`** — 289 named corners where car-free living is realistic (a frequent-transit stop with grocery, pharmacy, and ≥3 daily needs in a walk): `city`, `location`, `lat`, `lon`, `frequent_stops_nearby`, `daily_needs_nearby`.

**`car_free_cities.csv`** — car-free node counts by city: `city`, `car_free_nodes`, `frequent_transit_stops`, `pct_stops_car_free`.

**`pedestrian_safety_by_county.csv`** — where pedestrians die and on what road, for 953 counties (NHTSA FARS 2022–2024): `county_fips`, `county`, `state`, `ped_deaths_3yr`, `pct_deaths_on_surface_arterials`, `pct_deaths_after_dark`, and a road-type breakdown (`pct_arterial`, `pct_highway`, `pct_collector`, `pct_local`). Aggregated to county because tract-level crash counts are too sparse to be honest.

## Companion docs (paid bundle)
- `START_HERE.md` — what the file is, the honest skeptic's FAQ, and how to use it.
- `METHODOLOGY.md` — how every number is computed, with sources.
- `FRAMEWORKS_AND_STANDARDS.md` — the sourced standard + threshold for every dimension.
- `URBANISM_PRIMER.md` — the urbanist thinking behind each metric.
- `EXAMPLE_LLM_PROMPTS.md` — copy-paste prompts to get outcomes from this file with an LLM.
- `KEY_FINDINGS.md` — 22 cited findings you can reference or verify.
</content>
