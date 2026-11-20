# Concrete Pour Planner

## Purpose
A planning tool for a concrete pour. The user outlines a slab on satellite imagery (or types its dimensions), and the app computes how much concrete to order, how many truckloads it takes, how many trucks the batch plant needs to keep the pour going without a break, and when each truck should arrive.

## Users
A construction management student, superintendent, or project engineer planning a slab pour and calling in the order to the batch plant.

## Platform
- A single index.html with plain HTML, CSS, and JavaScript.
- Map: Leaflet 1.9.4 from unpkg, with Esri World Imagery tiles (https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}) and the required attribution. No API keys.

## Inputs
All numeric ranges are inclusive.
- Slab geometry has two modes, selected with a "Draw on map" / "Enter dimensions" switch. The initial slab is a 40 ft x 60 ft rectangle shown on the map, with the dimension fields set to match.
  - The initial mode is "Enter dimensions"; the active mode determines which geometry is used for calculations. Switching modes does not convert one geometry into the other.
  - Draw-on-map control: a "Draw slab" button starts drawing; click the corners and double-click (or click the first point) to finish. Corners can be dragged afterward. An outline must have at least three distinct corners, nonzero area, and no self-intersections.
  - Dimensions control: numeric length and width fields in feet, each from 1 to 50 ft. The rectangle is drawn on the map at the map center.
  - The map opens centered on BYU campus (40.2518, -111.6493) at zoom 18.
- Slab thickness: numeric field in inches, 1–24 in, default 6.
- Waste allowance: numeric field in percent, 0–60%, default 5%.
- Truck capacity: numeric field in cubic yards, 1–12 cy, default 10 cy.
- One-way travel time from the batch plant: numeric field in minutes, 0–300 min, default 20 min.
- Placement rate the crew can sustain: numeric field in cubic yards per hour, 1–250 cy/hr, default 30 cy/hr.
- Washout and reload time at the plant: numeric field in minutes, 0–100 min, default 10 min.
- Time the first truck should arrive: time-of-day control from 00:00 to 23:59, default 7:00 AM. Enter and display times in jobsite-local time without timezone conversion. Schedules may cross midnight; times on the following day are marked as next-day times.

## Calculations
- Slab area (ft^2): for typed dimensions, length x width. For a drawn outline with successive vertices given as (latitude, longitude), close the outline from the last vertex to the first, convert all coordinates to radians, and use
  A = R^2 x |sum over edges of (lon2 - lon1) x (2 + sin lat1 + sin lat2)| / 2
  with angles in radians and R = 6,378,137 m, converted to ft^2
  (1 m^2 = 10.7639 ft^2).
- Volume to order (cubic yards) =
  area x thickness / 12 / 27 x (1 + waste / 100), rounded up to the next 0.5 cy; an exact 0.5 cy increment is unchanged. Equivalently, order volume = ceil(2 x unrounded volume) / 2.
- Truckloads = volume / capacity, rounded up. Every load is full except the last.
- Full-load unload time, in minutes = 60 x capacity (cy) / placement rate (cy/hr). For the final partial load, unload time, in minutes = 60 x final load volume (cy) / placement rate (cy/hr).
- Let the first truck's arrival time mean its arrival at the jobsite. Loads arrive one full-load unload time apart, including the final partial load: load k arrives at first arrival + (k - 1) x full-load unload time.
- Truck cycle time, from a truck's jobsite arrival until its next jobsite arrival, = 2 x one-way travel time + full-load unload time + washout/reload time. The truck's first trip starts one-way travel time before its scheduled first arrival.
- Trucks needed in rotation = the truck cycle time / full-load unload time, rounded up, but no more than the number of loads. Loads are assigned to trucks in turn: truck 1, truck 2, and so on, then truck 1 again. This schedule assumes travel and turnaround take exactly their specified durations, with no delay buffer.
- Pour duration runs from the first truck's arrival until the last truck is empty: (number of loads - 1) x full-load unload time + final-load unload time.
- Keep time calculations at full precision. Round displayed clock times and durations to the nearest minute, after calculating the schedule; an exact half-minute rounds up.

## Outputs
- The slab drawn on the map with its area and order volume (including waste and 0.5 cy rounding) on a label.
- Summary cards: concrete to order (cy), truckloads, trucks in rotation, pour duration, and finish time.
- A truck schedule as a timeline (like a Gantt chart): one row per truckload, labeled with its load number and assigned truck number, showing travel to, unloading, travel back, and washout/reload in different colors on a jobsite-local clock-time axis. Segment times are available on hover, keyboard focus, and tap, so they are usable on touch screens.
- An "Order summary" box with a plain-text order for the batch plant (volume, number of loads, first jobsite arrival, and spacing between jobsite arrivals) and a "Copy" button. Give clear confirmation when copying succeeds and surface copy failures.
- Valid input changes update all outputs immediately. Invalid inputs show an error and do not silently leave stale results looking current.

## Look and feel
- Opens with an example slab already drawn near the center of the map, so it is useful the moment it opens.
- Clean and professional, like a commercial construction app: white cards with soft shadows, one accent color, a modern system font, large readable numbers on the summary cards.
- On desktop, map and inputs are on the left; summary, timeline, and order are on the right. At phone widths, use one column in this order: map and inputs, summary cards, timeline, order summary.
- Keep text and controls readable at phone widths, with visible keyboard focus and sufficient color contrast.
- A short hint tells the user what to do next, such as "Draw the slab on the map or enter its dimensions."

## Validation
1. Typed slab 40 ft x 60 ft x 6 in, 5% waste, 10 cy trucks, 20 min travel, 30 cy/hr, 10 min washout, first truck 7:00 AM:
   - Area 2,400 ft^2; 44.44 cy; with waste 46.67 cy; order 47.0 cy.
   - 5 truckloads (four of 10 cy and one of 7).
   - Unload time 20 min, so trucks arrive at 7:00, 7:20, 7:40, 8:00, and 8:20.
   - Truck cycle time 2 x 20 + 20 + 10 = 70 min, so 4 trucks in rotation.
   - Last truck holds 7 cy and unloads in 14 min, so the pour finishes at 8:34 AM. Pour duration 1 h 34 min.
2. Area: an outline with corners (40.2500, -111.6500), (40.2510, -111.6500), (40.2510, -111.6490), (40.2500, -111.6490) has an area of about 101,800 ft^2 (within 1%).
3. Typed slab 20 ft x 20 ft x 4 in, 0% waste, 10 cy trucks: 400 ft^2, 4.94 cy, order 5.0 cy, 1 truckload, 1 truck, pour duration 10 min. Unspecified inputs use their defaults, including 20 min travel, 30 cy/hr placement rate, 10 min washout/reload, and 7:00 AM first arrival; the 5 cy final load therefore unloads in 10 min.
4. Repeat example 1 with the first truck arriving at 11:50 PM. Arrivals are 11:50 PM, 12:10 AM, 12:30 AM, 12:50 AM, and 1:10 AM; arrivals after midnight and the 1:24 AM finish are marked as next-day times. The truck count and 1 h 34 min pour duration remain unchanged.
- Calculation boundary checks: pre-rounding volume of exactly 1.0 cy yields an order of 1.0 cy; 1.001 cy yields 1.5 cy. An order volume of exactly 20 cy with 10 cy truck capacity yields 2 loads. A 5 cy final load at a placement rate of 200/7 cy/hr takes exactly 10.5 min; it displays as 11 min, and if it arrives at 7:21 AM it finishes at 7:32 AM.

## Out of scope
- Reinforcement, finishing crews, pump trucks, weather, and costs.
- Live routing, traffic, route optimization, and actual order submission; the user supplies travel time, and the app only prepares copyable order text.
- Saving or loading plans and server-side storage. The app does not persist plans or provide an app-controlled export; the order can be copied to the clipboard.
- Offline operation; the app depends on the external Leaflet CDN and map tiles.

## Build stages
1. **Static page mockup**
   - Build the page layout in the single `index.html`, using a placeholder map and static sample inputs and results. Do not implement map interactions or calculations.
   - Test that the desktop two-column layout and the specified phone stacking order are clear, readable, and keyboard accessible.
2. **Map and geometry controls**
   - Add Leaflet and the specified imagery tiles, initial 40 ft x 60 ft rectangle, geometry mode switch, typed dimensions, map drawing, and draggable corners. Keep results static; do not add calculations.
   - Test map center, zoom, and attribution; rectangle updates from typed dimensions; drawing completion by either specified gesture; corner dragging; and mode switching without geometry conversion.
3. **Area and order-volume calculations**
   - Calculate typed and drawn slab area, volume including waste, and 0.5 cy order rounding. Update the map label and concrete-to-order output; leave truck and schedule results static.
   - Test the area and order-volume results in Validation 1–3, the exact and just-above rounding boundary checks, and invalid-input feedback.
4. **Loads and pour calculations**
   - Calculate load count and final load size, unload times, truck arrival spacing, truck cycle and rotation count, pour duration, and finish time.
   - Test Validation 1 and 3, including the 70-minute cycle, four-truck rotation, partial final load, and the cap of one truck for one load. Check that an exact truck-capacity multiple does not add a load.
5. **Schedule, order copy, and final interaction checks**
   - Render the truck timeline with load and truck identifiers, activity segments, and accessible times. Add the copyable order summary and copy success/failure feedback; complete responsive and accessibility checks.
   - Test timeline segments against Validation 1, next-day times in Validation 4, fractional-minute rounding, copy behavior, and timeline usability by hover, keyboard focus, and tap on desktop and phone.