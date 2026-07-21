# Key Findings

Findings drawn from this dataset and SafeStreets' related public-data layers. Use them to frame analysis or as citable stats. Each carries its geography and source.

> Full methodology with a citable DOI: https://doi.org/10.5281/zenodo.20506270. More analyses at safestreets.streetsandcommons.com/insights.

## The safety blind spot in the federal walkability index

- **The federal walkability index rates Las Vegas and Philadelphia as equally walkable. A pedestrian in Las Vegas is killed at a 56% higher rate.**
  EPA National Walkability Index: Las Vegas 11.74, Philadelphia 11.75. Pedestrian deaths per 100k per year: Las Vegas 3.72, Philadelphia 2.38.
  _US metro, population-weighted. Source: EPA Smart Location Database v3 + NHTSA FARS 2022-2024._
- **The reason: Las Vegas has more intersections per square mile than Philadelphia (132 vs 101), and the EPA index weights intersection density at one third.** Its suburbs are gridded, so the index rewards them. The grid is made of multi-lane arterials.
  _US metro. Source: EPA SLD v3 component D3B._
- **The EPA index contains no crash data of any kind.** It is built from street-network geometry and land-use mix only. It counts intersections; it cannot ask what happens when you cross one.
  _Source: EPA SLD v3 technical documentation._
- **Los Angeles ranks in the 75th percentile for walkable layout and the 31st for pedestrian safety, a gap of 44 points.**
  _US metro. Source: this dataset, `walk_safety_gap`._
- **Miami flatters itself more than anywhere else in America: +52.9.** Boston (-15.9) and Minneapolis (-21.4) are the reverse, safer than their street layout suggests.
  _US metro. Source: this dataset, `walk_safety_gap`._

## From this national sample

- **Walkable-and-affordable neighborhoods cluster in older industrial metros.** In this 6,000-block group sample, the 80 block groups that are both walkable and affordable are led by Chicago (13), Philadelphia (11), then Buffalo, St. Louis, Baltimore, and Pittsburgh. Walkable urban cores are mostly expensive, so the affordable-and-walkable ones concentrate in the Rust Belt. Source: EPA SLD v3 (Most Walkable) plus ACS 2020 to 2024 value-to-income at or below 3.0.
- **The most-walkable neighborhoods concentrate in a few big metros.** Los Angeles, New York, Chicago, San Francisco, Philadelphia, and Boston hold the largest counts of "most walkable" block groups in the sample. Walkable America is geographically concentrated. Source: EPA SLD v3.
- **By share, the Northeast and Pacific Northwest lead.** Among states with a meaningful sample, Oregon, Massachusetts, New York, Pennsylvania, and Washington have the highest share of block groups in the EPA "most walkable" tier. Source: EPA SLD v3.
- **Walkability spans the full national range.** The sample runs from 3.13 (deeply car-dependent) to 19.67 (among the most walkable in the country), median 12.54. Source: EPA SLD v3.

## National context (the full dataset)

- **Only about 11% of US households live in a 15-minute city, roughly 1 in 9.** Source: EPA Smart Location Database v3, top walkability tier.
- **A home in the most walkable US neighborhoods costs about 73% more per dollar earned than in the least walkable**, about 6.6 years of local income versus 3.8. Source: Zillow ZHVI (Apr 2026) joined to ACS 2020 to 2024 income, over EPA tiers.
- **Six metro areas hold about 41% of all US walkable neighborhoods. The top twelve hold about 53%.** Source: EPA Smart Location Database v3.
- **Only about 1% of US households live somewhere both walkable and affordable.** Source: EPA SLD v3 (Most Walkable) plus ACS 2020 to 2024 value-to-income at or below 3.0.
- **Arterial roads are about 9.5% of US road miles but account for roughly 62% of pedestrian deaths.** Per mile, arterials kill pedestrians at several times the rate their share of the network would predict. Source: NHTSA FARS 2022 to 2024.
- **About 95% of walkable US neighborhoods have at least some transit service.** Walkability and transit grew up together. Source: EPA SLD v3, transit proximity in the Most Walkable tier.

The complete national dataset (all 219,586 block groups, Zillow market prices and rents, curated opportunity lists, and the full research bank) is at https://sabarish0.gumroad.com/l/walkable-america?utm_source=kaggle
