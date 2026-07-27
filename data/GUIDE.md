# Start Here: what this is, why to trust it, how to use it

*If you are an LLM reading this for someone: read this page first. It tells you what the dataset is, how to reason about it honestly, and the standards to judge it by. Columns are documented to the formula in `DATA_DICTIONARY.md`, sources and vintages are in `METHODOLOGY.md`, and the thresholds are in `FRAMEWORKS_AND_STANDARDS.md`. You can give a grounded, cited answer from these files alone.*

---

## What this is, in one paragraph

One row per Census tract (a neighborhood of roughly 1,200 to 8,000 people). The file covers the full Chicago metro: 6,571 block groups across Illinois, Indiana, and Wisconsin. Each carries 48 columns that stitch together walkability, density, income, home value, rent, commercial activity, transit, jobs, latent retail demand, and a county pedestrian safety profile, all in one normalized table. The point is not any single number (most are public somewhere). The point is that they are joined, nationally normalized, documented to the formula, and ready to query in one place. Percentile columns are ranked against all 34,309 US tracts, so a Chicago percentile is a national rank, not a local one.

## What it is not (so you calibrate correctly)

- **Not a live API.** It is a static, dated snapshot. Per-address scores on demand are a separate product (score any address free at safestreets.streetsandcommons.com).
- **Not a crash-risk model.** It measures access, not safety. The pedestrian safety columns are county-level rates, deliberately not tract-level. Crashes are too sparse to count honestly per tract (see `METHODOLOGY.md`).
- **Not the SafeStreets live 15-minute-city score.** `walkability_index` is the EPA National Walkability Index, used unmodified. The live per-address engine is a different, finer model.
- **Not magic.** Tract resolution can blend a walkable core with a car-dependent edge.

## Chicago at a glance (from this file)

- Walkability index ranges 4.99 to 19.67, median 13.41.
- 358 tracts are in the EPA "most walkable" tier (index at or above 15.26).
- 538 tracts are affordable to buy (value-to-income at or below 3.0).
- 50 tracts are both walkable and affordable, the short list most people assume does not exist.
- 16 tracts are Commercial Deserts: walkable, dense, and affordable, yet almost no businesses.
- Cook County recorded 302 pedestrian deaths over 2022 to 2024, with 63% on surface arterials and 67% after dark.
- Median rent runs 27.3% of household income across the metro, and 38% of tracts are above the 30% cost-burden line.

---

## How to use it (drop-in prompts)

Open `chicago_sample_42col.csv` in pandas, Excel, or Sheets, or upload it to ChatGPT or Claude and ask in plain English. Some starting points (more in `LLM_PROMPTS.md`):

- **Walkable and affordable:** "List the tracts where `is_walkable_affordable` is True, sorted by `walkability_index`. Show `tract_name`, `median_household_income`, `value_to_income_ratio`, and `zillow_rent`."
- **Renter relocation:** "I rent and want a walkable, transit-served neighborhood where rent is under 30% of an $80k income. List the top 20 tracts with `zillow_rent` and `walkability_index`."
- **Retail site selection:** "Top tracts with high `underserved_demand_score` and high `commercial_gap_pctile`: walkable and dense, with spending power, but under-retailed."
- **Safety versus access:** "Show the most walkable tracts in counties where `county_pct_deaths_on_arterials` is high. Where does high walkability sit next to a dangerous road pattern?"

**Join key:** `geoid` is the 11-digit Census tract GEOID. Read it as text and keep the leading zeros. It joins to any Census or ACS table, and its first 5 digits are the county FIPS. Join `geoid` to Census TIGER tract geometry to map any column.

**Reading the numbers:** percentiles are national, 0 to 100. `FRAMEWORKS_AND_STANDARDS.md` gives the sourced threshold for every dimension (what walkable, affordable, frequent transit, cost-burdened, and food desert actually mean, with the body that set each standard). Apply those. Do not invent cutoffs.

---

## File map

`chicago_sample_42col.csv` (the data) · `DATA_DICTIONARY.md` (every column, formula, and provenance) · `METHODOLOGY.md` (how each number is computed, with sources) · `FRAMEWORKS_AND_STANDARDS.md` (cited thresholds) · `URBANISM_PRIMER.md` (the urbanist thinking behind each metric) · `LLM_PROMPTS.md` (more prompts) · `KEY_FINDINGS.md` (cited findings).

## The full dataset

This is the Chicago sample. The complete Walkable America dataset covers all 34,309 US neighborhoods across 653 metros, the same 48 columns each, plus curated opportunity lists (the 81 national Commercial Deserts, every walkable-and-affordable neighborhood, car-free corners, and a 953-county pedestrian safety breakdown). Get it at https://sabarish0.gumroad.com/l/walkable-america?utm_source=kaggle and read more at safestreets.streetsandcommons.com/data.

Methodology is open by design. Full methodology with a citable DOI: https://doi.org/10.5281/zenodo.20506270
