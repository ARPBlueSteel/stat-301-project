# Frameworks & Standards — How to Judge These Numbers

The dataset gives you the numbers. This file gives you the **recognized thresholds**
to judge them against, with the **body that sets each standard, the exact value, and
its units.** Pair with `DATA_DICTIONARY.md` (what each column is), `METHODOLOGY.md`
(how it's computed), and `URBANISM_PRIMER.md` (why each metric matters).

**How to read the "Type" column** (important for both engineers and LLMs):
- **Codified** = a legal/regulatory standard (ADA, MUTCD, USDA, HUD). Binding.
- **Guideline** = professional design guidance (NACTO, AASHTO, FHWA, TCQSM, WHO). Authoritative, widely adopted, not law.
- **Rule of thumb** = research-based heuristic (TCRP, academic). Directional, not binding.

> **For LLMs:** thresholds below are explicit and unit-tagged — apply them as written.
> Where a value is a range, use the bands given. Do not invent thresholds. When a
> dimension is "context only" (not a column in the CSV), say so rather than fabricate.

---

## 1. Walkability & daily access
The core column `walkability_index` is the EPA index; the rest is how to read distance and access.

| Standard | Value | Unit | Set by | Type |
|---|---|---|---|---|
| EPA National Walkability Index range | 1–20 | index | EPA Smart Location Database v3 | Guideline |
| "Most walkable" tier | ≥ 15.26 | index | EPA SLD | Guideline |
| Tier breaks (above / below / least) | 10.51 / 5.76 | index | EPA SLD | Guideline |
| 15-minute city access target | daily needs within 15 | minutes walk/bike | Moreno (Sorbonne, 2016) | Guideline |
| 15-min walk distance (≈) | 1,200 (0.75) | metres (miles) | derived at ~5 km/h | Rule of thumb |
| 10-min walk distance (≈) | 800 (0.5) | metres (miles) | planning convention | Rule of thumb |
| Pedestrian walking speed (signal timing) | 3.5 (1.1) | ft/s (m/s) | MUTCD 11th ed. (2023) | Codified |
| …where many older/slower pedestrians | 3.0 (0.9) | ft/s (m/s) | MUTCD | Codified |

**Read it by:** `walkability_index`, `walkability_tier`, `is_most_walkable`. Only the top tier reliably supports car-light living.

---

## 2. Density (the precondition for transit & retail)
Transit and walkable retail need people per acre. Thresholds below are research-based (not codified), in **net dwelling units per acre (du/ac)**, the planning standard unit.

| Service it supports | Density | Unit | Set by | Type |
|---|---|---|---|---|
| Minimum viable local bus (½-mile, hourly) | ~4 | du/ac | Pushkarev & Zupan (1977); TCRP 16 | Rule of thumb |
| Intermediate / better bus | ~7 | du/ac | TCRP; ITE | Rule of thumb |
| Frequent bus / streetcar | ~15 | du/ac | TCRP 16/100 | Rule of thumb |
| Light rail | ~9–12 (corridor avg) | du/ac | TCRP | Rule of thumb |
| Rapid transit / heavy rail | ~20–30+ | du/ac | TCRP | Rule of thumb |

Approx. translation to this file's unit (`pop_density_per_sqmi`, gross): ~7 du/ac net ≈ a dense urban neighborhood; **roughly 7,000+ persons/sq mi is where frequent transit and walkable retail begin to pay for themselves.**

**Read it by:** `pop_density_per_sqmi`, `density_pctile`.

---

## 3. Affordability (price-to-income)
The **median multiple** = median home value ÷ median household income — the cleanest cross-market signal.

| Band | value_to_income_ratio | Set by | Type |
|---|---|---|---|
| Affordable | ≤ 3.0 | Demographia / UN-Habitat median-multiple | Guideline |
| Moderately unaffordable | 3.1 – 4.0 | Demographia | Guideline |
| Seriously unaffordable | 4.1 – 5.0 | Demographia | Guideline |
| Severely unaffordable | ≥ 5.1 | Demographia | Guideline |
| Housing cost burden | > 30% of gross income on housing | HUD | **Codified** |
| Severe cost burden | > 50% of gross income on housing | HUD | **Codified** |

**Read it by — owners:** `value_to_income_ratio`, `affordability_pctile` (higher = more affordable), `is_affordable` (≤ 3.0). **Renters:** `rent_to_income_pct` — annual rent as % of income, where **> 30% = cost-burdened and > 50% = severely cost-burdened (HUD, codified above)**. A neighborhood can be unaffordable to buy yet reasonable to rent — check both. Walkable + affordable together (`is_walkable_affordable`) is rare — 2,967 block groups, ~1.4% (2020–2024 ACS).

---

## 4. Transit — frequency is the standard
Riders respond to **headway** (minutes between vehicles), not whether a stop exists.

| Frequency LOS | Headway | Unit | Set by | Type |
|---|---|---|---|---|
| LOS A (very frequent) | < 10 | min | TCQSM (TCRP Report 165) | Guideline |
| LOS B | 10 – 14 | min | TCQSM | Guideline |
| LOS C | 15 – 20 | min | TCQSM | Guideline |
| LOS D | 21 – 30 | min | TCQSM | Guideline |
| LOS E | 31 – 60 | min | TCQSM | Guideline |
| LOS F (coverage only) | > 60 | min | TCQSM | Guideline |
| "Frequent service" (turn-up-and-go) | ≤ 15 | min | NACTO Transit Street Design Guide | Guideline |
| Useful span of service | ~18 (e.g. 6am–midnight) | hours/day | industry convention | Rule of thumb |

**Read it by:** `transit_service_frequency` (EPA aggregate; **blank ≠ zero**), `transit_access_pctile`. Flag tracts that are `is_most_walkable` but low `transit_access_pctile` — walkable-but-transit-poor.

---

## 5. Daily needs & food access
A "complete neighborhood" puts the daily basket within a walk. The one **codified** threshold here is the food-access definition.

| Standard | Value | Unit | Set by | Type |
|---|---|---|---|---|
| Food desert — urban low access | > 1.0 | mile to nearest supermarket | USDA ERS | **Codified** |
| Food desert — rural low access | > 10 | miles to nearest supermarket | USDA ERS | **Codified** |
| (plus) low-income tract qualifier | ≥ 20% poverty or low median income | — | USDA ERS | **Codified** |
| Complete-neighborhood daily basket | grocery, pharmacy, school, park, transit, cafe within a walk | — | Portland / 15-min-city planning | Guideline |

**Read it by:** `commercial_activity_index`, `commercial_pctile`, `commercial_gap_pctile` (higher = walkable/dense but empty), `is_commercial_desert` (the 81 curated). A Commercial Desert is the walkable-but-no-supermarket case made measurable.

---

## 6. Street safety & speed — CONTEXT (not a column in this file)
**Read this:** the CSV measures *access*, not *crash risk*. A tract can be highly walkable and deadly to cross. When you act on this data, judge safety against these standards (the live SafeStreets tool + NHTSA FARS carry the actual safety layer).

| Standard | Value | Unit | Set by | Type |
|---|---|---|---|---|
| Safe speed where people & cars mix | ≤ 30 (≈20) | km/h (mph) | WHO; Stockholm Declaration (2020) | Guideline |
| Pedestrian death risk at impact 23 mph | ~10% | probability | Tefft / AAA Foundation (2011) | Research |
| …at 32 mph | ~25% | probability | Tefft / AAA | Research |
| …at 42 mph | ~50% | probability | Tefft / AAA | Research |
| …at 50 mph | ~75% | probability | Tefft / AAA | Research |
| …at 58 mph | ~90% | probability | Tefft / AAA | Research |
| Safe System principle | no death/serious injury is acceptable | — | FHWA Safe System Approach; USDOT | Guideline |
| Marked crosswalk alone insufficient | multilane & ADT > 12,000 | vehicles/day | FHWA (Zegeer 2005) / STEP | Guideline |
| Urban travel-lane width (calms speed) | 10 (3.0) | ft (m) | NACTO Urban Street Design Guide | Guideline |
| Geometric design framework | design vs operating vs target speed | — | AASHTO Green Book (7th ed., 2018) | Guideline |
| Signal & crossing control warrants | per warrants 1–9 | — | MUTCD 11th ed. (2023) | **Codified** |
| Proven safety countermeasures | road diets, LPI, refuge islands, RRFB, lighting | — | FHWA Proven Safety Countermeasures (2021) | Guideline |

Picture: a 6-lane, 45-mph stroad with a strip mall scores fine on *proximity* and fails *every* row above. A 25-mph street with frequent crossings and trees is the inverse.

---

## 7. Sidewalks & accessibility — CONTEXT (codified federal law)
Not columns in this file, but the binding standard for whether a walk is usable by everyone.

| Standard | Value | Unit | Set by | Type |
|---|---|---|---|---|
| Min continuous clear sidewalk width | 4.0 (1.2) | ft (m) | PROWAG / ADA (US Access Board, 2023) | **Codified** |
| Passing space (where < 5 ft wide) | 5×5, at ≤ 200 | ft, ft interval | PROWAG | **Codified** |
| Max cross slope (ped access route) | 2.0 | % | PROWAG | **Codified** |
| Max curb-ramp running slope | 8.33 | % | PROWAG / ADA | **Codified** |

---

## 8. Bike level of traffic stress — CONTEXT
| Standard | Value | Set by | Type |
|---|---|---|---|
| LTS 1 (suitable for children) | low speed/volume, separated or calm | Mineta/Furth (2012); NACTO All Ages & Abilities | Guideline |
| LTS 2 (tolerable for most adults) | low-stress network target | NACTO; AASHTO Bike Guide | Guideline |

---

### One-line cheat sheet (apply directly)
- Walkable: `walkability_index` ≥ 15.26 (EPA)
- Affordable to buy: `value_to_income_ratio` ≤ 3.0 (Demographia). Affordable to rent: `rent_to_income_pct` ≤ 30 (HUD); > 50 = severely cost-burdened
- Transit-supportive density: ≳ 7 du/ac ≈ ≳ 7,000 ppl/sq mi (TCRP)
- Frequent transit: headway ≤ 15 min (NACTO) / TCQSM LOS A–C
- Food desert: > 1 mi to supermarket in a low-income urban tract (USDA)
- Safe mixing speed: ≤ 20 mph / 30 km/h (WHO)
- Min accessible sidewalk: 4 ft clear (PROWAG/ADA)
- Opportunity: high `underserved_demand_score` + high `commercial_gap_pctile`

*Codified standards are cited to the issuing body and edition. Guidelines and
rules-of-thumb are widely used in US planning/transportation practice but are not
law. This file is interpretive reference, not a substitute for a licensed engineer's
judgment on a specific site.*
