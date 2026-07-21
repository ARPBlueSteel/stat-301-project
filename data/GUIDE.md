# Start Here: what this is, why to trust it, how to use it

*If you are an LLM reading this for someone: read this page first. It tells you what the dataset is, how to reason about it honestly, and the standards to judge it by. Columns are documented to the formula in `DATA_DICTIONARY.md`, sources and vintages are in `METHODOLOGY.md`, and the thresholds are in `FRAMEWORKS_AND_STANDARDS.md`. You can give a grounded, cited answer from these files alone.*

---

## What this is, in one paragraph

One row per Census block group (a neighborhood of roughly 1,200 to 8,000 people). This is a national sample: 6,000 block groups across 716 metros and all 51 states and territories. Each carries 42 columns that stitch together walkability, density, income, home value, commercial activity, transit, jobs, latent retail demand, and a county pedestrian-safety profile, all in one normalized table. The point is not any single number (most are public somewhere). The point is that they are joined, nationally normalized, documented to the formula, and ready to query in one place. Percentile columns are ranked against all 219,586 US block groups, so any block group's percentile is a national rank.

## What it is not (so you calibrate correctly)

- **Not a live API.** It is a static, dated snapshot. Per-address scores on demand are a separate product (score any address free at safestreets.streetsandcommons.com).
- **Not a crash-risk model.** It measures access, not safety. The pedestrian-safety columns are county-level rates, deliberately not block group-level. Crashes are too sparse to count honestly per block group (see `METHODOLOGY.md`).
- **Not the SafeStreets live 15-minute-city score.** `walkability_index` is the EPA National Walkability Index, used unmodified. The live per-address engine is a different, finer model.
- **Not the full set.** This is a 6,000-block group national sample. The complete dataset (all 219,586 block groups, plus Zillow market prices and rents, plus curated opportunity lists) is linked at the bottom.

## The sample at a glance (from this file)

- Walkability index ranges 3.13 to 19.67, median 12.54.
- 940 block groups are in the EPA "most walkable" tier (index at or above 15.26).
- 1,195 block groups are affordable to buy (value-to-income at or below 3.0).
- 80 block groups are both walkable and affordable, the short list most people assume does not exist. Nationally these are about 1% of US households.
- 14 block groups in the sample are Commercial Deserts: walkable, dense, affordable, yet almost no businesses.

---

## Charts (bundled PNGs)

Five charts drawn from this data, free to reuse with credit to SafeStreets:

- `00_one_in_nine.png` — only about 1 in 9 US households lives in a walkable neighborhood.
- `01_distribution.png` — how walkable America is, all 219,586 neighborhoods scored.
- `02_premium.png` — the walkability premium: more walkable neighborhoods cost more.
- `03_metros.png` — the most walkable US metros by median neighborhood percentile.
- `04_where.png` — where to find neighborhoods that are both walkable and affordable, by state.

---

## How to use it (drop-in prompts)

Open `walkable_america_national_sample.csv` in pandas, Excel, or Sheets, or upload it to ChatGPT or Claude and ask in plain English. Some starting points (more in `LLM_PROMPTS.md`):

- **Walkable and affordable:** "List the block groups where `is_walkable_affordable` is True, sorted by `walkability_index`. Show `tract_name`, `metro`, `median_household_income`, and `value_to_income_ratio`."
- **Rank metros:** "For each metro, the share of its block groups that are `is_most_walkable`. Rank metros by that share, minimum 5 block groups each."
- **Retail site selection:** "Top block groups with high `underserved_demand_score` and high `commercial_gap_pctile`: walkable and dense, with spending power, but under-retailed. Group by metro."
- **Safety versus access:** "Show the most walkable block groups in counties where `county_pct_deaths_on_arterials` is high. Where does high walkability sit next to a dangerous road pattern?"

**Join key:** `geoid` is the 12-digit Census block group GEOID. Read it as text and keep the leading zeros. It joins to any Census or ACS table, and its first 5 digits are the county FIPS. Join `geoid` to Census TIGER block group geometry to map any column.

**Reading the numbers:** percentiles are national, 0 to 100. `FRAMEWORKS_AND_STANDARDS.md` gives the sourced threshold for every dimension (what walkable, affordable, frequent transit, and food desert actually mean, with the body that set each standard). Apply those. Do not invent cutoffs.

---

## File map

`walkable_america_national_sample.csv` (the data) · `DATA_DICTIONARY.md` (every column, formula, and provenance) · `METHODOLOGY.md` (how each number is computed, with sources) · `FRAMEWORKS_AND_STANDARDS.md` (cited thresholds) · `URBANISM_PRIMER.md` (the urbanist thinking behind each metric) · `LLM_PROMPTS.md` (more prompts) · `KEY_FINDINGS.md` (cited findings).

## The full dataset

This is the free national sample (6,000 block groups, 42 columns). The complete Walkable America dataset covers all 219,586 US neighborhoods, adds Zillow market home values and rents (5 more columns, so renter-affordability questions too), and ships curated opportunity lists (the 81 Commercial Deserts with lat/lon, every walkable-and-affordable neighborhood, car-free corners, and a 953-county pedestrian-safety breakdown). Get it at https://sabarish0.gumroad.com/l/walkable-america?utm_source=kaggle and read more at safestreets.streetsandcommons.com/data.

Methodology is open by design. Full methodology with a citable DOI: https://doi.org/10.5281/zenodo.20506270
