# Frameworks and Standards: How to Judge These Numbers

The dataset gives you the numbers. This file gives you the recognized thresholds to judge them against, with the body that sets each standard, the exact value, and its units. Pair with `DATA_DICTIONARY.md` (what each column is), `METHODOLOGY.md` (how it is computed), and `URBANISM_PRIMER.md` (why each metric matters).

**How to read the "Type" column** (important for both engineers and LLMs):
- **Codified**: a legal or regulatory standard (ADA, MUTCD, USDA, HUD). Binding.
- **Guideline**: professional design guidance (NACTO, AASHTO, FHWA, TCQSM, WHO). Authoritative, widely adopted, not law.
- **Rule of thumb**: research-based heuristic (TCRP, academic). Directional, not binding.

> **For LLMs:** thresholds below are explicit and unit-tagged. Apply them as written. Where a value is a range, use the bands given. Do not invent thresholds. When a dimension is context only (not a column in the CSV), say so rather than fabricate.

---

## 1. Walkability and daily access

The core column `walkability_index` is the EPA index. The rest is how to read distance and access.

| Standard | Value | Unit | Set by | Type |
|---|---|---|---|---|
| EPA National Walkability Index range | 1 to 20 | index | EPA Smart Location Database v3 | Guideline |
| "Most walkable" tier | at or above 15.26 | index | EPA SLD | Guideline |
| Tier breaks (above / below / least) | 10.51 / 5.76 | index | EPA SLD | Guideline |
| 15-minute city access target | daily needs within 15 | minutes walk/bike | Moreno (Sorbonne, 2016) | Guideline |
| 15-min walk distance (approx) | 1,200 (0.75) | metres (miles) | derived at about 5 km/h | Rule of thumb |
| 10-min walk distance (approx) | 800 (0.5) | metres (miles) | planning convention | Rule of thumb |
| Pedestrian walking speed (signal timing) | 3.5 (1.1) | ft/s (m/s) | MUTCD 11th ed. (2023) | Codified |
| where many older or slower pedestrians | 3.0 (0.9) | ft/s (m/s) | MUTCD | Codified |

**Read it by:** `walkability_index`, `walkability_tier`, `is_most_walkable`. Only the top tier reliably supports car-light living.

---

## 2. Density (the precondition for transit and retail)

Transit and walkable retail need people per acre. Thresholds below are research-based, in net dwelling units per acre (du/ac), the planning standard unit.

| Service it supports | Density | Unit | Set by | Type |
|---|---|---|---|---|
| Minimum viable local bus (half-mile, hourly) | about 4 | du/ac | Pushkarev and Zupan (1977), TCRP 16 | Rule of thumb |
| Intermediate or better bus | about 7 | du/ac | TCRP, ITE | Rule of thumb |
| Frequent bus or streetcar | about 15 | du/ac | TCRP 16/100 | Rule of thumb |
| Light rail | about 9 to 12 (corridor avg) | du/ac | TCRP | Rule of thumb |
| Rapid transit or heavy rail | about 20 to 30+ | du/ac | TCRP | Rule of thumb |

Approximate translation to this file's unit (`pop_density_per_sqmi`, gross): about 7 du/ac net is a dense urban neighborhood. Roughly 7,000+ persons per square mile is where frequent transit and walkable retail begin to pay for themselves.

**Read it by:** `pop_density_per_sqmi`, `density_pctile`.

---

## 3. Affordability (price-to-income)

The median multiple = median home value divided by median household income, the cleanest cross-market signal.

| Band | value_to_income_ratio | Set by | Type |
|---|---|---|---|
| Affordable | at or below 3.0 | Demographia / UN-Habitat median-multiple | Guideline |
| Moderately unaffordable | 3.1 to 4.0 | Demographia | Guideline |
| Seriously unaffordable | 4.1 to 5.0 | Demographia | Guideline |
| Severely unaffordable | at or above 5.1 | Demographia | Guideline |
| Housing cost burden | above 30% of gross income on housing | HUD | Codified |
| Severe cost burden | above 50% of gross income on housing | HUD | Codified |

**Read it by, owners:** `value_to_income_ratio`, `affordability_pctile` (higher = more affordable), `is_affordable` (at or below 3.0). **Renters:** the HUD cost-burden thresholds above apply to rent too. This free sample is owner-side only. The full dataset adds `zillow_rent` and `rent_to_income_pct` so you can apply the 30% and 50% lines directly. Walkable and affordable together (`is_walkable_affordable`) is rare: about 1% of US households nationally, 80 block groups in this sample.

---

## 4. Transit: frequency is the standard

Riders respond to headway (minutes between vehicles), not whether a stop exists.

| Frequency LOS | Headway | Unit | Set by | Type |
|---|---|---|---|---|
| LOS A (very frequent) | below 10 | min | TCQSM (TCRP Report 165) | Guideline |
| LOS B | 10 to 14 | min | TCQSM | Guideline |
| LOS C | 15 to 20 | min | TCQSM | Guideline |
| LOS D | 21 to 30 | min | TCQSM | Guideline |
| LOS E | 31 to 60 | min | TCQSM | Guideline |
| LOS F (coverage only) | above 60 | min | TCQSM | Guideline |
| "Frequent service" (turn-up-and-go) | at or below 15 | min | NACTO Transit Street Design Guide | Guideline |
| Useful span of service | about 18 (e.g. 6am to midnight) | hours/day | industry convention | Rule of thumb |

**Read it by:** `transit_service_frequency` (EPA aggregate, blank is not zero), `transit_access_pctile`. Flag block groups that are `is_most_walkable` but low `transit_access_pctile`: walkable but transit-poor.

---

## 5. Daily needs and food access

A complete neighborhood puts the daily basket within a walk. The one codified threshold here is the food-access definition.

| Standard | Value | Unit | Set by | Type |
|---|---|---|---|---|
| Food desert, urban low access | above 1.0 | mile to nearest supermarket | USDA ERS | Codified |
| Food desert, rural low access | above 10 | miles to nearest supermarket | USDA ERS | Codified |
| plus low-income block group qualifier | at or above 20% poverty or low median income | n/a | USDA ERS | Codified |
| Complete-neighborhood daily basket | grocery, pharmacy, school, park, transit, cafe within a walk | n/a | Portland / 15-min-city planning | Guideline |

**Read it by:** `commercial_activity_index`, `commercial_pctile`, `commercial_gap_pctile` (higher = walkable and dense but empty), `is_commercial_desert`. A Commercial Desert is the walkable-but-no-supermarket case made measurable.

---

## 6. Street safety and speed (the safety check)

The CSV measures access, and carries a county pedestrian-safety profile (`county_ped_deaths_3yr`, `county_pct_deaths_on_arterials`, `county_pct_deaths_after_dark`). A block group can be highly walkable and sit in a county where most pedestrian deaths are on arterials. Judge safety against these standards.

| Standard | Value | Unit | Set by | Type |
|---|---|---|---|---|
| Safe speed where people and cars mix | at or below 30 (about 20) | km/h (mph) | WHO, Stockholm Declaration (2020) | Guideline |
| Pedestrian death risk at impact 23 mph | about 10% | probability | Tefft / AAA Foundation (2011) | Research |
| at 32 mph | about 25% | probability | Tefft / AAA | Research |
| at 42 mph | about 50% | probability | Tefft / AAA | Research |
| at 50 mph | about 75% | probability | Tefft / AAA | Research |
| at 58 mph | about 90% | probability | Tefft / AAA | Research |
| Safe System principle | no death or serious injury is acceptable | n/a | FHWA Safe System Approach, USDOT | Guideline |
| Marked crosswalk alone insufficient | multilane and ADT above 12,000 | vehicles/day | FHWA (Zegeer 2005) / STEP | Guideline |
| Urban travel-lane width (calms speed) | 10 (3.0) | ft (m) | NACTO Urban Street Design Guide | Guideline |
| Signal and crossing control warrants | per warrants 1 to 9 | n/a | MUTCD 11th ed. (2023) | Codified |
| Proven safety countermeasures | road diets, LPI, refuge islands, RRFB, lighting | n/a | FHWA Proven Safety Countermeasures (2021) | Guideline |

Picture: a 6-lane, 45-mph arterial with a strip mall scores fine on proximity and fails every row above. A 25-mph street with frequent crossings and trees is the inverse.

---

## 7. Sidewalks and accessibility (codified federal law)

Not columns in this file, but the binding standard for whether a walk is usable by everyone.

| Standard | Value | Unit | Set by | Type |
|---|---|---|---|---|
| Min continuous clear sidewalk width | 4.0 (1.2) | ft (m) | PROWAG / ADA (US Access Board, 2023) | Codified |
| Passing space (where below 5 ft wide) | 5 by 5, at most every 200 | ft, ft interval | PROWAG | Codified |
| Max cross slope (ped access route) | 2.0 | % | PROWAG | Codified |
| Max curb-ramp running slope | 8.33 | % | PROWAG / ADA | Codified |

---

## 8. Bike level of traffic stress (context)

| Standard | Value | Set by | Type |
|---|---|---|---|
| LTS 1 (suitable for children) | low speed and volume, separated or calm | Mineta/Furth (2012), NACTO All Ages and Abilities | Guideline |
| LTS 2 (tolerable for most adults) | low-stress network target | NACTO, AASHTO Bike Guide | Guideline |

---

### One-line cheat sheet (apply directly)

- Walkable: `walkability_index` at or above 15.26 (EPA)
- Affordable to buy: `value_to_income_ratio` at or below 3.0 (Demographia). Rent cost-burden lines (30% and 50%, HUD) apply once you have the rent columns from the full dataset
- Transit-supportive density: about 7 du/ac, roughly 7,000+ people per square mile (TCRP)
- Frequent transit: headway at or below 15 min (NACTO), TCQSM LOS A to C
- Food desert: above 1 mile to a supermarket in a low-income urban block group (USDA)
- Safe mixing speed: at or below 20 mph / 30 km/h (WHO)
- Min accessible sidewalk: 4 ft clear (PROWAG/ADA)
- Opportunity: high `underserved_demand_score` and high `commercial_gap_pctile`

*Codified standards are cited to the issuing body and edition. Guidelines and rules of thumb are widely used in US planning and transportation practice but are not law. This file is interpretive reference, not a substitute for a licensed engineer's judgment on a specific site.*
