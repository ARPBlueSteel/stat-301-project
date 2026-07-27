# Key Findings — SafeStreets Walkability Research

27 findings derived from this dataset and SafeStreets' related public-data layers. Use them to frame analysis or as citable stats. `published` = verified and used publicly; `high` confidence unless noted.

> Source links and full methodology live at safestreets.streetsandcommons.com/insights.


## The safety blind spot in the federal walkability index

- **The federal walkability index rates Las Vegas and Philadelphia as equally walkable. A pedestrian in Las Vegas is killed at a 56% higher rate.**  
  EPA National Walkability Index: Las Vegas 11.74, Philadelphia 11.75. Pedestrian deaths per 100k per year: Las Vegas 3.72, Philadelphia 2.38.  
  _US metro · source: EPA Smart Location Database v3 + NHTSA FARS 2022–2024, population-weighted · high_
- **The reason: Las Vegas has more intersections per square mile than Philadelphia (132 vs 101), and the EPA index weights intersection density at one third.** Its suburbs are gridded, so the index rewards them. The grid is made of multi-lane arterials.  
  _US metro · source: EPA SLD v3 components (D3B) · high_
- **The EPA index contains no crash data of any kind.** It is built from street-network geometry and land-use mix only. It counts intersections; it cannot ask what happens when you cross one.  
  _source: EPA SLD v3 technical documentation · high_
- **Los Angeles is the largest example. It ranks in the 75th percentile for walkable layout and the 31st for pedestrian safety, a gap of 44 points.**  
  _US metro · source: this dataset, `walk_safety_gap` · high_
- **Miami flatters itself more than anywhere else in America: +52.9.** Built like you could walk, deadly if you try. Boston (−15.9) and Minneapolis (−21.4) are the reverse — safer than their street layout suggests.  
  _US metro · source: this dataset, `walk_safety_gap` · high_

## Walkability

- **Only 11.35% of US households live in a 15-minute city. About 1 in 9.**  
  _US national · source: EPA Smart Location Database v3 (top NatWalkInd tier) · high_
- **A home in the most walkable US neighborhoods costs 73% more per dollar earned than in the least walkable: about 6.6 years of local income vs 3.8.**  
  _US national · source: Zillow ZHVI (Apr 2026) joined to ACS 2020–2024 income, over EPA tiers · high_
- **Six metro areas hold 40.7% of all US walkable neighborhoods; the top twelve hold 53.2%.**  
  _US national · source: EPA Smart Location Database v3 · high_
- **About 1.4% of US block groups are both walkable and affordable (2,967 of 219,586); they cluster in older industrial and Rust Belt metros — Chicago (326), Philadelphia (269), Pittsburgh (157), New York (139), St. Louis (99), and Baltimore (90) lead.**  
  _US national, Rust Belt concentration · source: EPA SLD v3 (Most Walkable) + ACS 2020–2024 value-to-income ≤ 3.0 · high_
- **95.56% of walkable US neighborhoods have at least some transit service. Walkability and transit grew up together.**  
  _US national · source: EPA SLD v3 (transit proximity D4A in Most-Walkable tier) · high_
- **Central Jakarta has almost no sidewalks in OpenStreetMap despite being one of Southeast Asia's densest cores: 78 sidewalks for 7,676 street segments.**  
  _Jakarta, Indonesia (Sudirman/Thamrin core, 2.5km radius) · source: OpenStreetMap via Overpass, 2026-06-09; real-world figure 8.2m sidewalk per 100m road (Jakarta city data, 2021) · high_

## Retail

- **81 US neighborhoods are walkable, dense, and affordable yet have almost no businesses. The bones are there; the shops are missing.**  
  _US national · source: EPA SLD v3 + business/POI data · high_
- **Chicago has 16 of America's 81 walkable, dense, business-starved neighborhoods, the most of any metro. Top spot: Douglass Park (4,430 people per sq mi, six-figure median income, almost no storefronts).**  
  _US national, Chicago hero · source: EPA Smart Location DB v3 + Census + business density · verify_
- **Of the 8 Pittsburgh tracts the EPA dataset flagged as 'walkable, dense, business-empty,' only 2 areas hold up under ground-truth review as genuine residential-density-with-thin-retail candidates: Brighton Heights (California Ave, called underbuilt in the neighborhood plan) and Carrick (Brownsville Rd, where vacancy is the issue, not absence).**  
  _Pittsburgh, PA · source: EPA Smart Location DB v3 + per-tract ground-truth web research 2026-05-28 · high_

## Transit

- **Share of metro population within a 5-minute walk of transit running every 15 min or better ranges from 35% in Chicago to 2.9% in Atlanta. A 12-fold gap inside one country.**  
  _8 major US transit metros · source: GTFS schedules across 8 metros · high_
- **Most US transit is coverage, not frequency. A stop on the map does not give independence if the bus comes a couple times an hour.**  
  _US · source: SafeStreets transit-frequency analysis (frequency vs coverage distinction) · high_
- **In Richmond, only about 18% of GRTC stops run every 15 min or better midday; the Church Hill route is closer to every 27 minutes.**  
  _Richmond, VA (GRTC) · source: GRTC published GTFS, Jan 2024 service (Mobility Database) · verify_
- **8 of the 16 2026 World Cup stadiums sit within a short walk of everyday rapid transit. 2 have no rail to the venue at all, including Arlington, the largest US city with no public transit.**  
  _North America (US, Canada, Mexico host cities) · source: Host-committee and transit-agency 2026 plans (TransLink, Sound Transit, METRO Houston, SEPTA, MARTA, Metrolinx, VTA, Tren Ligero, Metrorrey, LA Metro, NJ Transit, MBTA, Brightline, ConnectKC) · high_

## Peds

- **Arterial roads are 9.5% of US road miles but cause 62.5% of pedestrian deaths.**  
  _US national · source: NHTSA FARS 2022-2024 · high_
- **Per mile, arterials kill pedestrians at 6.6x the rate their share of the network would predict.**  
  _US national · source: NHTSA FARS 2022-2024 · high_
- **40% of US pedestrian deaths happen at one location type: mid-block on an arterial road, not at intersections.**  
  _US national · source: NHTSA FARS 2022-2024 · high_
- **26,458 pedestrians killed on US roads, 2022-2024.**  
  _US national · source: NHTSA FARS 2022-2024 · verify_
- **62% of US pedestrian deaths with a known road name are on surface arterials, the streets with crossings and speed, not freeways.**  
  _US national · source: NHTSA FARS 2022-2024 (PER_TYP=5, INJ_SEV in 4,6) · verify_
- **America's deadliest pedestrian streets, 2022-2024: US-90 New Orleans (21), SR-50 Orlando (17), US-19 Pasco County FL (16), Western Ave Los Angeles (14), W Thomas Rd Phoenix (14), Colfax Ave Aurora (13), Westheimer Rd Houston (11), Broad St Philadelphia (10).**  
  _US national, per-metro available · source: NHTSA FARS 2022-2024 · high_
- **In Houston, 325 pedestrians were killed 2022-2024. Westheimer Rd is the deadliest corridor with 19 deaths, more than the next three combined. 51% of Houston pedestrian deaths are on surface arterials and roughly 80% happen after dark; median victim age 43.**  
  _Houston, TX · source: NHTSA FARS 2022-2024 · verify_

## Street-Design

- **Pedestrian survival drops from about 90% at 30 km/h to under 10% at 60 km/h. Speed is the lever.**  
  _global / physics of impact · source: Impact-speed survival research, summarized in Street Design Determines Survival · high_

## Mobility

- **60% of US trips of one mile or less are made by car. A 15-minute walk becomes a calculation that ends with a key in an ignition.**  
  _US national · source: National Household Travel Survey 2022 · high_
