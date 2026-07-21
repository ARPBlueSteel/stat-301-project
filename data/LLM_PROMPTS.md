# Get Outcomes from This Dataset with an LLM

This file is built to be ingested by ChatGPT, Claude, or any LLM with file upload or code execution. Upload `walkable_america_national_sample.csv`, paste a prompt below, and get an answer in seconds. Every column referenced is defined in `DATA_DICTIONARY.md`. Upload that too for best results.

## Setup prompt (paste first)

> I am uploading `walkable_america_national_sample.csv` and `DATA_DICTIONARY.md`. Each row is a US Census block group with walkability, affordability, commercial activity, transit, demographics, and a county pedestrian-safety profile. Percentiles are national. Use the data dictionary for column meanings and `FRAMEWORKS_AND_STANDARDS.md` for the thresholds. Answer with tables, cite the `geoid` and `tract_name`, and show your filters. Read `geoid` as text. Do not invent columns.

## Walkable and affordable

- "List the block groups where `is_walkable_affordable` is True, sorted by `walkability_index`. Show `tract_name`, `metro`, `state`, `median_household_income`, and `value_to_income_ratio`."
- "For each state, what share of its block groups are `is_walkable_affordable`? Rank states by that share."
- "Name the single most walkable affordable neighborhood in each of the 10 largest metros in this sample, with the numbers."

## Rank metros and states

- "For each metro with at least 5 block groups, the share that are `is_most_walkable`. Rank metros by that share."
- "Which states have the highest median `walkability_index`? Which have the lowest?"
- "Compare median `value_to_income_ratio` for Most walkable versus Least walkable block groups. Quantify the walkability premium."

## Site selection and real estate

- "List the 25 block groups with the highest `underserved_demand_score` that are `is_walkable_affordable` = True and `population` above 3,000. Show metro, state, income, and `commercial_gap_pctile`."
- "I open neighborhood grocery stores. Rank the top block groups in Texas that are walkable and dense but have low `commercial_pctile` and high `commercial_gap_pctile`. Group by metro."
- "Find walkable block groups (`walkability_index` at or above 15) where `value_to_income_ratio` is between 3 and 4.5 and population density is in the top 20% nationally: demand but not yet priced out."
- "Summarize the block groups where `is_commercial_desert` = True by metro and state."

## Safety and access

- "Which highly walkable block groups (`is_most_walkable` = True) sit in counties with the highest `county_pct_deaths_on_arterials`? Where does walkable access meet a dangerous road pattern?"
- "Rank counties in this sample by `county_ped_deaths_3yr`, and show their `county_pct_deaths_on_arterials` and `county_pct_deaths_after_dark`."

## Research, policy, public health

- "What percent of total `population` in this sample lives in Most walkable block groups? In `is_walkable_affordable` block groups?"
- "Within a chosen metro, map the relationship between `walkability_index` and `affordability_pctile`. Are walkable neighborhoods less affordable?"
- "Identify block groups that are highly walkable but have low `transit_access_pctile`: walkable but transit-poor."

## Journalism and content

- "Give me 5 surprising, defensible statistics from this dataset, each with the exact filter and block group count behind it."
- "Which metro has the most walkable-and-affordable neighborhoods in this sample, and what do they have in common?"

## Tips

- `geoid` is text with leading zeros. Tell the LLM to read it as a string. Its first 5 digits are the county FIPS.
- Percentiles are national, 0 to 100. Filter on them to compare against the whole country.
- Booleans are the literal strings True and False in the CSV.
- Join `geoid` to Census TIGER block group geometry to map any column.
- This is a 6,000-block group sample. Phrase national-share questions as "in this sample" unless you intend the full population.

---

The full Walkable America dataset (all 219,586 US block groups, plus Zillow market prices and rents for renter-affordability questions, curated opportunity lists, and a larger prompt library) is at https://sabarish0.gumroad.com/l/walkable-america?utm_source=kaggle
