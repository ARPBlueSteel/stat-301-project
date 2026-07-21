# Data Dictionary: Walkable America (national sample)

One row = one Census block group (a neighborhood of roughly 1,200 to 8,000 people).
File: `walkable_america_national_sample.csv`, 6,000 block groups across 716 metros and 51 states and territories.

> Percentiles are national, 0 to 100 (100 = highest in the country for that measure) unless noted. The ranking is computed against all 219,586 US block groups, so a block group at the 90th percentile is in the top 10% nationally.

---

## How to read provenance (start here)

Every column is tagged so you know exactly what you are holding. Perplexity, ChatGPT, and a careful human all ask the same first question: which numbers are real measurements and which did you compute? This answers it per column.

**Raw / Derived flag (R/D):**
- **R**: Raw pass-through from a public source (may be unit-converted only). Income and home value ship as reported by the Census, not projected or inflated.
- **D**: Derived, computed from raw inputs. Formula in the Derived-field formulas section.
- **D-pct**: A national percentile rank (0 to 100) of a base column.
- **D-flag**: A boolean from a published threshold.
- **ID**: Identifier or join key (raw Census geography).

Of the 42 columns: 4 are identifiers, 14 are raw measurements, and 24 are derived. The derived fields are the product: the join, the normalization, and the decision shortcuts. Nothing here is a black box. Every derived value traces to a raw input and a stated formula.

---

## Master column table

| Column | Type | R/D | Source, Vintage | Description |
|---|---|---|---|---|
| `geoid` | string | ID | Census TIGER 2023 | 12-digit Census block group GEOID (state, county, block group). Leading zeros preserved, read as text. Join key to any Census or ACS data, and to TIGER block group geometry for mapping. |
| `tract_name` | string | ID | Census TIGER 2023 | Human-readable Census name (block group; county; state). |
| `metro` | string | ID | Census CBSA 2023 | Metro area the block group belongs to. |
| `state` | string | ID | Census 2023 | 2-letter state abbreviation. |
| `walkability_index` | float | R | EPA SLD v3 (2021) | EPA National Walkability Index, 1 to 20. Higher = more walkable street network and destination access. Used unmodified. |
| `walkability_pctile` | float | D-pct | derived from EPA SLD v3 | National percentile of `walkability_index`. |
| `county_ped_death_rate_per_100k` | float | **D** | NHTSA FARS 2022-2024 + block-group population | Annual pedestrian deaths per 100,000 residents in the parent county. `county_ped_deaths_3yr / 3 / county population x 100,000`. Blank where FARS has no county record. |
| `ped_safety_pctile` | float | D-pct | derived (FARS) | National percentile of pedestrian safety. **100 = safest** (lowest death rate). Inverted so higher is always better, like every other percentile here. |
| `safe_walk_score` | float | **D** | **SafeStreets composite** | **The SafeStreets Walk Safety Score, 0-100.** `0.7 x walkability_pctile + 0.3 x ped_safety_pctile`. Answers what the EPA index cannot: is this place laid out for walking, and will walking there get you killed. The 70/30 weighting matches the Street Safety weight in SafeStreets' live 15-minute-city score. |
| `safe_walk_tier` | string | **D** | SafeStreets composite | Label for `safe_walk_score`: Walkable and safe (>=80), Above average (>=60), Average (>=40), Below average (>=20), Car-dependent or dangerous (<20). |
| `walk_safety_gap` | float | **D** | **SafeStreets composite** | `walkability_pctile - ped_safety_pctile`. **How much a place flatters itself.** Positive = built like you could walk, dangerous if you try (Las Vegas +48.6, Miami +52.9 at metro scale). Negative = safer than its street layout suggests (Boston -15.9, Minneapolis -21.4). This column exists in no other dataset. |
| `pop_density_per_sqmi` | float | R | EPA SLD v3, Census | Population density, persons per square mile (approx.). |
| `density_pctile` | float | D-pct | derived | National percentile of density. |
| `median_household_income` | int | R | ACS 2020 to 2024 (5-yr) | Median household income, USD. Census ACS 2024 5-year release, as reported, not inflation-adjusted. Blank where ACS suppressed the estimate. |
| `income_pctile` | float | D-pct | derived | National percentile of income. |
| `median_home_value` | int | R | ACS 2020 to 2024 (5-yr) | Median home value, USD, ACS owner-reported, as reported. Block group-level. Runs roughly 15 to 20% below market because ACS is self-reported and topcoded. The full dataset adds current Zillow market value. |
| `value_to_income_ratio` | float | D | derived (ACS) | `median_home_value / median_household_income`. At or below 3 = affordable, at or above 5 = severely unaffordable (Demographia median multiple). |
| `affordability_pctile` | float | D-pct | derived | National percentile, inverted: higher = more affordable (lower value-to-income ratio). |
| `commercial_activity_index` | float | D | composite, from EPA SLD employment | Relative density of commercial and business activity, scaled 0 to 1. See formulas. |
| `commercial_pctile` | float | D-pct | derived | National percentile of commercial activity. |
| `commercial_gap_pctile` | float | D | composite | Higher = more underserved: walkable and dense but commercially empty. The Commercial Deserts signal. |
| `transit_service_frequency` | float | R | EPA SLD v3 (D4c) | EPA aggregate transit service frequency per square mile. Blank where EPA had no record. Blank is not zero. |
| `transit_access_pctile` | float | D-pct | derived | National percentile of transit access. |
| `retail_emp_density` | float | R | EPA SLD v3 (D1C5_RET) | Retail employment density. |
| `service_emp_density` | float | R | EPA SLD v3 (D1C5_SVC) | Service employment density. |
| `entertainment_emp_density` | float | R | EPA SLD v3 (D1C5_ENT) | Entertainment employment density. |
| `retail_jobs` | int | R | EPA SLD v3 (E5_Ret) | Retail-sector jobs in the block group. |
| `service_jobs` | int | R | EPA SLD v3 (E5_Svc) | Service-sector jobs. |
| `entertainment_jobs` | int | R | EPA SLD v3 (E5_Ent) | Entertainment-sector jobs. |
| `population` | int | R | EPA SLD v3 (ACS base) | Total population (EPA SLD `TotPop`, ACS-derived). |
| `households` | int | R | EPA SLD v3 (ACS base) | Total households (EPA SLD `HH`). |
| `demand_income_pctile` | float | D | derived | Spending-power component of latent retail demand = `income_pctile`. |
| `demand_afford_pctile` | float | D | derived | Affordability component of latent retail demand = `affordability_pctile`. |
| `underserved_demand_score` | float | D | composite (transparent) | 0 to 100: unmet demand for daily-needs businesses in a walkable, dense, affordable but empty block group. The site-selection signal. Exact formula below. |
| `walkability_tier` | string | D-flag | derived (EPA cutoffs) | `Most walkable` (at or above 15.26), `Above average` (10.51 to 15.25), `Below average` (5.76 to 10.50), `Least walkable` (below 5.76). |
| `is_most_walkable` | bool | D-flag | derived | `walkability_index` at or above 15.26 (EPA "most walkable" tier). |
| `is_affordable` | bool | D-flag | derived | `value_to_income_ratio` above 0 and at or below 3.0. |
| `is_walkable_affordable` | bool | D-flag | derived | Both walkable and affordable. About 1% of US households nationally, 80 block groups in this sample. |
| `is_commercial_desert` | bool | D-flag | curated | One of the curated Commercial Deserts: walkable, dense, affordable, yet almost no businesses. The full dataset carries the national set of 81 with lat/lon. |
| `county_ped_deaths_3yr` | int | R | NHTSA FARS 2022 to 2024 | Pedestrian fatalities in this block group's county, 2022 to 2024. County-level, not block group-precise (crashes too sparse to count per block group honestly, see methodology). |
| `county_pct_deaths_on_arterials` | int | D | rate, from FARS 2022 to 2024 | Share of those county deaths on surface arterials, %. The single strongest danger pattern. |
| `county_pct_deaths_after_dark` | int | D | rate, from FARS 2022 to 2024 | Share of those county deaths between 6pm and 6am, %. |

---

## Derived-field formulas

Exact formula for every derived column. The entire income to affordability to demand chain is reproducible from this file, with no hidden weighting.

**Simple arithmetic (fully reproducible from this file):**
- `value_to_income_ratio` = `median_home_value / median_household_income`
- `walkability_tier` = step function on `walkability_index`: at or above 15.26 is Most walkable, at or above 10.51 is Above average, at or above 5.76 is Below average, else Least walkable (EPA SLD cutoffs)
- `is_most_walkable` = `walkability_index` at or above 15.26
- `is_affordable` = `value_to_income_ratio` above 0 and at or below 3.0
- `is_walkable_affordable` = `is_most_walkable` AND `is_affordable`
- `is_commercial_desert` = block group is one of the manually-reviewed Commercial Deserts
- `county_pct_deaths_on_arterials` = arterial pedestrian deaths / all pedestrian deaths in the county (FARS 2022 to 2024) x 100
- `county_pct_deaths_after_dark` = pedestrian deaths 6pm to 6am / all pedestrian deaths in the county x 100

**Percentiles (`*_pctile`)** = national rank of the base column across all 219,586 US block groups, scaled 0 to 100 (average-tie rank / N x 100). `affordability_pctile` is inverted (100 = lowest value-to-income ratio = most affordable). All others run in the natural direction (100 = highest).

**The demand model (fully transparent):**
- `demand_income_pctile` = `income_pctile`, spending-power proxy.
- `demand_afford_pctile` = `affordability_pctile`, affordability proxy.
- `underserved_demand_score` = `clip( 0.6 x demand + 0.4 x (100 minus commercial_pctile), 0, 100 )`, where
  `demand = 0.35 x density_pctile + 0.25 x walkability_pctile + 0.20 x demand_income_pctile + 0.20 x demand_afford_pctile`.
  High = a block group with the residents and foot-traffic to support daily-needs retail (dense, walkable, with spending power) but low commercial supply. The site-selection signal. Pairs with `commercial_gap_pctile`.

**Composites (upstream EPA enrichment, inputs and direction exact):**
- `commercial_activity_index`: relative density of commercial activity, built from EPA SLD employment and establishment counts, scaled 0 to 1. Higher = more active commercial fabric.
- `commercial_gap_pctile`: the gap between walkability and density on one side and `commercial_activity_index` on the other. Higher = walkable and dense but commercially empty.

---

## Refresh schedule

| Source | Columns it feeds | Cadence |
|---|---|---|
| US Census ACS 5-yr | `median_household_income`, `median_home_value` and the ratios, percentiles, demand built on them | Annual (Dec). Current base is 2020 to 2024 |
| NHTSA FARS | `county_ped_deaths_3yr`, `county_pct_deaths_on_arterials`, `county_pct_deaths_after_dark` | Annual |
| EPA Smart Location Database v3 | `walkability_index`, density, transit, jobs, population, households | Irregular (EPA, v2 2014 to v3 2021) |

---

## What the full dataset adds

This national sample carries 42 columns. The complete Walkable America dataset adds:
- all 219,586 US block groups (this file is a 6,000-block group sample),
- 5 Zillow columns: `zillow_home_value` (ZHVI market price), `zillow_zcta`, `zillow_value_to_income_ratio`, `zillow_rent` (ZORI), and `rent_to_income_pct` (HUD cost-burden above 30%), so you can answer renter-affordability and survey-versus-market questions,
- curated opportunity lists: the 81 national Commercial Deserts with lat/lon, every walkable-and-affordable neighborhood, car-free corners, and a 953-county pedestrian-safety breakdown.

https://sabarish0.gumroad.com/l/walkable-america?utm_source=kaggle

Open methodology with a citable DOI: https://doi.org/10.5281/zenodo.20506270
