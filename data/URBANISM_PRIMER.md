# The Thinking Behind the Data

This dataset is not an arbitrary index. Every column is an attempt to measure something the great urbanists spent careers describing: what makes a neighborhood work. Here is the lineage, so you know what you are really looking at, and so the numbers carry the weight of a century of urban thought.

---

## The daily-needs neighborhood: `walkability_index`, `walkability_tier`

The oldest idea in the file. Clarence Perry's Neighborhood Unit (1929) put everything a household needs within a five-minute walk of an elementary school. Carlos Moreno's 15-Minute City (Sorbonne, 2016) is the modern restatement: work, shops, school, care, and leisure reachable in fifteen minutes on foot or bike. The EPA Walkability Index is, in effect, a national measurement of how close each neighborhood comes to that ideal, built from street connectivity, destinations, and transit.

## What makes a street alive: `commercial_activity_index`, `commercial_gap_pctile`, `pop_density_per_sqmi`

Jane Jacobs, The Death and Life of Great American Cities (1961), the book that ended top-down urban renewal. Her four generators of diversity: mixed primary uses, short blocks, buildings of varying age, and sufficient density. Her "eyes on the street": busy, mixed streets police themselves. A Commercial Desert in this dataset is a Jacobs failure made measurable, density and walkable bones but stripped of the mixed uses that give a street life. That is why `commercial_gap_pctile` exists.

## The walk has to be worth taking: why a high index alone is not enough

Jeff Speck, Walkable City (2012), the General Theory of Walkability: a walk must be useful, safe, comfortable, and interesting. Miss any one and people drive. Use `walkability_index` to find where things are close, then use `FRAMEWORKS_AND_STANDARDS.md` to ask whether the walk is also safe and comfortable. Proximity is necessary, not sufficient.

## Build for people at 5 km/h: the comfort dimension

Jan Gehl, Life Between Buildings and Cities for People: "First life, then spaces, then buildings. The other way around never works." Cities measured at eye level and human scale, not from a planner's helicopter. The walking-comfort layer (tree canopy, slope, heat, live in the address tool) is Gehl's concern: is this somewhere a person actually wants to be on foot.

## Why car-dependence happens: the low end of the index

Donald Shoup, The High Cost of Free Parking (2005): minimum-parking mandates quietly spread cities out, subsidize driving, and price out the density walkability needs. Enrique Penalosa: "An advanced city is not one where even the poor use cars, but one where even the rich use public transport." The Least walkable block groups are largely what these forces produced.

## Streets are social space, and traffic severs it: the safety caveat

Donald Appleyard, Livable Streets (1981): his San Francisco study found residents of lightly-trafficked streets reported about three times as many friends as those on heavy-traffic streets. Speed and volume are not just safety numbers. They decide whether a street is a community or a sewer for cars. This is why the standards file insists: walkable is not the same as safe. The county pedestrian columns in this file are the first measurable check on that.

## Walkability is priced in: `value_to_income_ratio`, `affordability_pctile`

Christopher Leinberger, Foot Traffic Ahead, documents large rent premiums for walkable-urban places. Joe Cortright, Walking the Walk (2009), found each Walk Score point adds measurable home value. The headline of this dataset, that homes in the most walkable US neighborhoods cost about 73% more per dollar earned, is that literature measured nationally. Walkability is not a soft amenity. The market has already priced it. The full dataset extends this to renting, with Zillow rent and a HUD cost-burden column, since most walkable-urban residents rent rather than own.

---

## The canon (if you want to go deeper)

- Jane Jacobs, The Death and Life of Great American Cities (1961)
- Jeff Speck, Walkable City (2012)
- Jan Gehl, Cities for People (2010)
- Donald Shoup, The High Cost of Free Parking (2005)
- Donald Appleyard, Livable Streets (1981)
- Kevin Lynch, The Image of the City (1960), legibility, how we read a place
- William H. Whyte, The Social Life of Small Urban Spaces (1980), how people actually use space
- Christopher Alexander, A Pattern Language (1977), the patterns of good places
- Carlos Moreno, The 15-Minute City (2016)

The numbers in this file are a measurement attempt at what these thinkers described. Read them with that in mind and you are not analyzing a spreadsheet. You are locating, at national scale, the neighborhoods they spent their lives arguing for.
