# Validation

This folder contains hand calculations and test cases. The results below were
checked in the browser against the calculation functions in the root
`index.html`; each numeric-input case also updated and checked the rendered
summary and schedule. No application code was changed.

The spec currently has four numbered validation examples (including the
midnight repeat) and additional calculation boundary checks. All were tested.

## Validation 1: 40 ft x 60 ft slab

Inputs: 6 in thickness, 5% waste, 10 cy capacity, 20 min one-way travel,
30 cy/hr placement, 10 min washout/reload, first arrival 7:00 AM.

| Output | Expected | Actual |
| --- | --- | --- |
| Area | 2,400 ft^2 | 2,400 ft^2 |
| Concrete before waste | 44.44 cy | 44.444444 cy |
| Volume after 5% waste, before order rounding | 46.67 cy | 46.666667 cy |
| Order volume (up to 0.5 cy) | 47.0 cy | 47.0 cy |
| Loads | 5: four 10 cy, one 7 cy | 5: four 10 cy, one 7 cy |
| Full-load arrival spacing | 20 min | 20 min |
| Truck cycle / rotation | 70 min / 4 trucks | 70 min / 4 trucks |
| Jobsite arrivals | 7:00, 7:20, 7:40, 8:00, 8:20 AM | 7:00, 7:20, 7:40, 8:00, 8:20 AM |
| Truck assignments | 1, 2, 3, 4, 1 | 1, 2, 3, 4, 1 |
| Final unload / finish | 14 min / 8:34 AM | 14 min / 8:34 AM |
| Pour duration | 1 h 34 min | 1 h 34 min |

## Validation 2: polygon area

For corners (40.2500, -111.6500), (40.2510, -111.6500),
(40.2510, -111.6490), and (40.2500, -111.6490):

| Output | Expected | Actual |
| --- | --- | --- |
| Polygon area | About 101,800 ft^2, within 1% | 101,804.20 ft^2; 0.0041% above 101,800 |

## Validation 3: 20 ft x 20 ft slab

Inputs: 4 in thickness, 0% waste, 10 cy capacity. Unspecified values use
defaults: 20 min one-way travel, 30 cy/hr placement, 10 min washout/reload,
and 7:00 AM first arrival.

| Output | Expected | Actual |
| --- | --- | --- |
| Area | 400 ft^2 | 400 ft^2 |
| Concrete before waste | 4.94 cy | 4.938272 cy |
| Volume after waste, before order rounding | 4.94 cy | 4.938272 cy |
| Order volume | 5.0 cy | 5.0 cy |
| Loads / final load | 1 load / 5 cy | 1 load / 5 cy |
| Full-load arrival spacing | 20 min | 20 min |
| Truck cycle / rotation | 70 min / 1 truck (capped by one load) | 70 min / 1 truck |
| Jobsite arrival | 7:00 AM | 7:00 AM |
| Final unload / finish | 10 min / 7:10 AM | 10 min / 7:10 AM |
| Pour duration | 10 min | 10 min |

## Validation 4: repeat Validation 1 at 11:50 PM

| Output | Expected | Actual |
| --- | --- | --- |
| Order volume / loads / rotation | 47.0 cy / 5 / 4 trucks | 47.0 cy / 5 / 4 trucks |
| Jobsite arrivals | 11:50 PM, then 12:10 AM, 12:30 AM, 12:50 AM, 1:10 AM next day | Same; arrivals after midnight marked next day |
| Pour duration / finish | 1 h 34 min / 1:24 AM next day | 1 h 34 min / 1:24 AM (next day) |

The truck assignments remain 1, 2, 3, 4, 1, and the final 7 cy load unloads
from 1:10 to 1:24 AM next day.

## Calculation boundary checks

| Check | Expected | Actual |
| --- | --- | --- |
| Round pre-waste-adjusted volume 1.0 cy | 1.0 cy | 1.0 cy |
| Round pre-waste-adjusted volume 1.001 cy | 1.5 cy | 1.5 cy |
| Loads for exactly 20 cy at 10 cy capacity | 2 | 2 |
| Unload 5 cy at 200/7 cy/hr | 10.5 min, displayed as 11 min | 10.5 min; displayed duration rounds to 11 min |
| Finish after that unload starting 7:21 AM | 7:32 AM (half-minute rounds up) | 7:32 AM |

All expected results matched the actual calculation outputs. No calculation
errors were found.

## Satellite Map Area Comparison

Add one row per building after measuring its footprint in both the Concrete
Pour Planner and Google Maps. Keep both measurements in the same area units.

| Building location | Area measured in Concrete Pour Planner | Area measured in Google Maps | Percent difference |
| --- | --- | --- | --- |
| [Building/location] | [Planner area] | [Google Maps area] | [Percent difference] |

Calculate percent difference as:

`|Planner area - Google Maps area| / ((Planner area + Google Maps area) / 2) x 100%`
