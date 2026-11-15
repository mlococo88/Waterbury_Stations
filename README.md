# Platform Geometry Engine

A single-file, offline HTML tool for laying out high-level rail station platforms. It computes top-of-platform elevations along the platform from the top-of-rail profile, then works down through the supporting structure to report every elevation needed to build it: beam seats, bearing plates, cap or pier wall tops, micropile cutoffs, pedestal and footing elevations.

Open `index.html` in any modern browser, or use the hosted copy at https://mlococo88.github.io/Waterbury_Stations/. No install, no server. Everything except KaTeX typesetting in the Calculations tab works offline.

The tool was developed for the Waterbury line stations (Beacon Falls and Derby are the worked examples), starting from parts of an earlier `Bridge_Geometry.html` tool.

---

## How the model works

The chain of control runs in one direction:

```
horizontal alignment ─┐
top-of-rail profile ──┼─► rail elevations at a station ─► platform edge ─► platform surface
superelevation (cant) ┘                                                        │
                                                                               ▼
               beams ─► bearings (sole + pad + masonry plate) ─► cap / pier wall top
                                                                               │
                         ┌─────────────────────────────────────────────────────┤
                         ▼                                                     ▼
          cap beam on micropiles                              pier wall on spread footing
          (bottom of cap, pile cutoffs)                       (footing from grade, excavation)
                         │                                                     │
                         └──────────────► canopy pedestal ◄────────────────────┘
                                          (top from cap; bottom on pile cap or footing)
```

Everything is stored in decimal feet internally. Stationing, feet-inches and inch inputs are display and parsing conventions only.

---

## Tabs

| Tab | What it holds |
|---|---|
| **Track** | Stationing format, POB, horizontal alignment, top-of-rail profile, superelevation, plan and profile views |
| **Platform** | Platform extent and side, edge control (H and V offsets), cross slope, width, tactile strip, platform profile mode, curve setback, tolerances, typical section |
| **Structure** | Deck panel, cap mode, bearing assembly, beam lines, pier type, cap/pier wall, spread footing, canopy pedestal, pedestal pile cap, micropiles, proposed grade, support layout and table, section through a support |
| **Platform Output** | Checks, top-of-platform table at an interval, cross section at any station, validation against known points |
| **Support Output** | Framing plan, support elevation table, beam seats, micropile layout, beam spans, longitudinal profile |
| **Ramps & Stairs** | Ramps and stairs tied into the platform back edge or ends, built as chains of segments; plan, profile, ADA checks, and ramp walls with footings (elevation view and tables) |
| **Heat Map** | Platform surface unrolled along the track, coloured by elevation, deviation from target height above rail, or running slope; contours and high/low points |
| **3D** | Orbitable 3D model of the whole structure with ground |
| **Calculations** | Full derivation of every elevation at a chosen support |
| **Notes** | Free-text project notes, saved with the platform |

---

## Track

### Stationing
Selectable between **100 ft blocks** (`100+00.00`, roadway) and **1000 ft blocks** (`42+400.00`, railroad). Switching reformats the station fields in place without changing any geometry. The setting saves with the project and is restored before stations are read, so a reload cannot misread them.

### Horizontal alignment
Built from a POB (station, bearing, northing, easting) and a list of elements:

- **Tangent**
- **Curve + spirals** — the usual railroad case. One row takes turn direction, radius, circular arc length, and a spiral length at each end; it expands internally into spiral–curve–spiral. Set either spiral length to 0 to omit it.
- **Curve only** and **Spiral only** for anything that doesn't fit the pattern.

Turn direction is a **Right/Left** choice (looking up-station); radii are always entered positive. This replaced an earlier signed-radius convention that caused the Beacon Falls curve to turn the wrong way.

**Curves given as** Radius, Degree of curve (chord definition, `R = 50 / sin(D/2)`), or Degree of curve (arc definition, `R = 5729.578 / D`). Railroad plans use chord definition — Beacon Falls D 3°30'00" gives R 1637.28 exactly.

Each compound curve shows a derived detail line: TS/SC/CS/ST stations, total length, Δ of the arc and total, D or R equivalent, spiral angle θs, throw *p*, offset *k*, and tangent distance T.

Spirals are clothoids integrated numerically. A spiral is fully defined by R and Ls.

**Verification (Beacon Falls):** POB 93+931.11, N5°54'W; tangent 225.81; curve-left R 1637.28, Lc 294.37, Ls 164.36 / 151.63. Computed SC and CS coordinates match the plan sheet within 0.02 ft. Δ arc reproduces the sheet's 10°18'05". (The sheet's last callout is labelled "SC 94+767.23" but is the ST; the app gives ST at 94+767.28 and its coordinates differ by about 3 ft — worth confirming against survey.)

### Top-of-rail profile
VPIs with parabolic vertical curves. Grades are editable directly and move the next VPI. The profile can be referenced to the track centreline, low rail, high rail, left rail or right rail.

### Superelevation
Cant control points in inches, interpolated linearly between rows. **Positive = right rail higher**, looking up-station. Rail elevations are derived from the profile reference and the cant across the gauge (default 4'-8½").

### Plan view
North-up or **track across the page**, with a north arrow that rotates with the view and a scale bar. Shows track, rails, platform, and cap footprints at true size. A check flags a platform whose limits fall outside the alignment.

---

## Platform

- **Extent** — either set the begin/end stations directly, or **derive** them from the outer supports plus the cantilevers (the stations become read-only and computed).
- **Side** — left or right of track, looking up-station. Every section drawing mirrors accordingly.
- **Edge control** — horizontal setback from ℄ track (default 5'-7") and height above rail (default 4'-0"), measured above the near rail, low rail or ℄.
- **Cross slope** — default 1%, draining away from or toward the track.
- **Platform profile** — *follow the rail* (constant offset, exact everywhere) or *independent* VPIs with the achieved height above rail checked against tolerance. "Seed from rail" builds platform VPIs at the limits and every support.
- **Curve setback** — optional automatic car-swing setback: mid-car inswing `Lt²/8R` on the inside of a curve, end-overhang outswing on the outside. Car body roll from cant is not modelled.
- **Tolerances** — max cross slope, running slope, vertical and horizontal gap, min/max height above rail.

---

## Structure

### Superstructure
- **Deck panel** thickness in inches (top of platform to top of beam).
- **Beam lines** with an AISC W-shape library (W12–W24; W18×50 default). Picking a section sets the depth and gives true flange/web dimensions for drawing. Custom depth is also allowed.

### Bearings and masonry plates
Bearing assembly = sole plate + elastomeric pad (constant) + masonry plate (varies).

The **anchor beam** has its masonry plate fixed (default **front beam at 1.5"**). That fixes the cap top, and every other beam's plate is solved from the cross slope. Anchor can also be the rear beam or whichever needs least. A **minimum plate anywhere** (default 0.5") is checked. Plates can be rounded to a fabrication increment (default 1/16"), with the resulting effect on top of platform reported.

**Cap mode:** *level* (one elevation, plates vary) or *sloped parallel to the platform* (constant plates).

### Pier type
One setting for the whole platform:

**Cap beam on micropiles**
- Cap depth, thickness along track, front-face offset, width.
- Micropile lines with offsets from cap front face, platform edge or ℄ track. Top of pile = bottom of cap + embedment (default 1'-0"). Casing diameter default 10".
- Pile tips are deliberately **not** modelled — they come from the geotechnical design. Piles are drawn as a short broken stub and left out of the tables.

**Pier wall on spread footing**
- Pier wall top = the same elevation a cap would have; nothing above changes.
- Bottom of footing = grade − burial (default 3.5 ft); top = bottom + thickness (default 2 ft).
- Footing extent = from the outermost face of the wall or pedestal on each side, plus a projection (default 1'-6") all round.
- Pedestal runs down to the top of the footing; no separate pile cap.
- Granular fill (default 2 ft) gives bottom of excavation, drawn with 1:1 side slopes.
- Micropile and pile cap inputs are hidden in this mode.

### Canopy pedestal
- Offset from platform edge or ℄ track; width default 2'-8".
- Top of pedestal = cap (or wall) top at the back face + rise (default 1'-0"). Column base adds grout pad (2") and base plate (1¾").
- **Pedestal pile cap** (cap-beam mode only): 4'-0" × 4'-0" × 2'-10", two micropiles at 1'-10". Located by whichever is deeper of (a) burial below grade (default 3.5 ft to bottom) and (b) minimum pedestal height (default 4.83 ft, top of pedestal to top of pile cap). Reports which governed.

### Proposed grade
A default grade plus a per-support override in the support table. Drives pile cap and footing elevations, cover, cap exposure, and the 3D ground surface.

### Support layout
- First and last support stations, count, and **cantilever at each end**. The cantilever boxes and first/last stations are bound both ways.
- **Generate supports** rebuilds the table (keeps names and overrides if the count is unchanged).
- **Shift supports to cantilevers** moves existing supports so the outer two sit at the cantilever positions, preserving names, grades and overrides.
- Supports are **normal to the track baseline** by default (skew locked to zero); unlocking allows skew.
- The readout reports both the planned layout and what is actually in the table, and flags a mismatch.

### Hidden inputs
Inputs that can't affect the answer are hidden: pile/pile cap inputs in spread-footing mode, pedestal inputs when the pedestal is off, pile inputs when micropiles are off, the anchor selector for a sloped cap, car length/truck centers when curve setback is off. Hidden values are kept.

---

## Outputs

### Checks
Platform within the alignment; cross slope; height above rail; running slope; horizontal and vertical gap to car; end cantilevers vs adjacent span; anchor plate and computed plate range; proposed grade presence; spread footing extent and wall height.

### Tables (CSV export where noted)
- Top of platform along the platform at an interval, with beam-line and back elevations and optional coordinates (CSV)
- Support elevation table: track, platform, cap/wall, plates per beam, top of pile, pedestal, pile cap or footing (CSV)
- Beam seats: every bearing component per beam per support
- Micropile layout: offsets, northing/easting, top of pile, embedment (CSV)
- Beam spans including cantilevers, with mid-span deviation from the platform surface

### Drawings
All drawings are true scale with pan/zoom.

- **Sections** (typical, at any station, through a support) with dimension chains, leaders and elevation callouts modelled on the HDR typical section. Labels are placed by a collision-avoiding pass with priority (dimensions → elevations → notes). Sections mirror for a left-side platform. A Dimensions toggle turns annotation off.
- **Framing plan**: platform, cap beams or footings, beam lines, micropiles, pedestal pile caps, cantilever callouts at both ends.
- **3D**: painter's-algorithm renderer, no external library. Layers for platform, tactile strip, beams (true wide flanges), cap beams/footings, micropiles, pedestals, canopy columns, ramps and stairs, ramp and stair walls with footings, track, ground (interpolated from per-support grades, adjustable opacity), and support labels. Station window, vertical exaggeration, pile stub length.

### Ramps & Stairs
Each ramp or stair is a **chain** of segments that starts where it ties into the platform and works downhill:

- **Tie-in** — the platform back edge at a station (heading up-station, down-station or away from the track), or the begin/end of the platform. The top elevation is read live from the platform surface, so ramps follow any change to the platform.
- **Segments** — landing (length), ramp run (length and slope 1:x), turn (90° or 180°; toward/away from the track, or up-/down-station), stair flight (risers, tread depth, tread count; leave the riser blank to solve it so the flight lands on grade). A 90° turn pivots about the corner square formed by the last width of the landing before it, so an L-shaped entry (a landing straight back from the platform, then the ramp along the track) is modelled directly.
- **Checks** — ramp slope 1:12, 30" max rise per run, 5' landings (a landing straight after a turn counts as part of the corner), 3' clear width; risers 4"–7", treads ≥ 11". A plan elevation can be entered beside each segment to compare against the drawings.
- **Drawings** — plan along the track with footprints, walls, footings, walking line and elevations; profile along the walking line (ramps vertically exaggerated up to 10×, stairs true scale).

**Ramp walls** — cast-in-place walls along both sides of every landing and run:

- Top of wall = adjacent walking surface + curb (default 8"). Wall thickness default 1'-2" (also spaces the two runs at a 180° turn).
- End walls across a 180° turning landing, across the far end of a 90° corner (lining up with the next run's outside wall), and optionally across the bottom of the last run. At a 90° corner the wall on the turn side stops where the new run's wall picks up. The side of the top landing facing the platform is open.
- Strip footing centred under the wall, default **3'-0" × 1'-6"** (11" toes, SP-214); the footing also runs a toe (11") past each end of the wall, so footings close at corners. **Footing bottom** either *level* (a set elevation, or frost depth below the lowest grade) or *follows grade* (frost depth below a straight-line grade from the top of the ramp to the bottom).
- **Low grade** (ramps and stairs) sets the bottom of footing: BOF = low grade − depth below grade (default 3'-6"). Left blank, it is the lowest grade entered at the piers next to the ramp or stair (piers within its station range plus the nearest one beyond each end), else the default grade. A bottom-of-footing elevation entered directly overrides it. Stair support wall footings are always level.
- Collinear pieces merge into one wall; at each break the top follows the higher adjacent surface.
- Walls meet square at corners: the wall that turns the corner runs to the outside face of the other, with no overhang.
- **Column pads** — where a column lands on a wall, the top of wall is level over a pad. Each pad is placed by station and distance behind the platform back edge (it finds the wall under it), with a length and either a set elevation, a depth below the platform, or blank (level at the higher end, stepping down to the sloping top at the low end). A pad within 6" of a wall end runs to the end.
- **Columns on the pads** — each pad can carry a column (HSS, default 8×8). Grout pad and base plate thicknesses default to the platform's (Structure tab) and can be set per ramp; Seymour uses a 1" grout pad. Outputs: bottom of base plate (BOBP = top of pad + grout) and top of base plate in the pad table and a *Columns on the walls* table (station, offset, coordinates, wall), the grout / plate / column stub and BOBP on the wall elevation, the column at true size in plan, and grout, plate and column in 3D (*Canopy columns* layer).
- **Entry walls under the platform** — for a landing straight back from the platform, the side walls can start a set distance in front of the back edge.

**Stair support walls** — one strip wall across the top of each stair, under the column carrying the stair. Top = platform − 1.125 ft (7" FRP + 4½" clear + 1" base plate + 1" grout, SP-312), length and grade at the wall as inputs, centreline 7½" behind the back edge (on SV-1) for back-edge stairs, at the platform end for end stairs.

**Outputs** — walls in plan with footing outlines; a developed true-scale elevation of any wall, dimensioned: top of wall at every end, break and pad, slope of each sloping stretch, curb, a vertical chain at each end (exposed height, cover to footing, footing thickness, overall height) with TOW/grade/TOF/BOF elevations, a horizontal chain of every stretch (pads, sloped, level) and the overall length; walls that meet or cross it are shown end-on as a dashed outline of their stem and footing, labelled, in feet and inches to the nearest ⅛" (top of wall, walking surface or platform, grade, top and bottom of footing, height at every break); a summary table and a per-wall table with centreline coordinates. Checks flag a wall top less than the curb above grade, a footing with less than frost cover, and a footing bottom sloping more than 10%.

The 3D view paints in three layers (wholly below grade, the ground and anything crossing it, wholly above grade) so buried footings always read as below the semi-transparent ground; ramp walls are split at grade for this. Ramps, stairs and walls show in the 3D view (layers *Ramps & stairs* and *Ramp & stair walls*: ramp slabs, stair treads on stringers, walls with their level pads, and footings). They are not yet in the framing plan, heat map or Calculations tab. They save with the platform (Library / Export JSON).

### Columns on spread footings
Free-standing columns (for example canopy columns on SV-2, off the piers and ramp walls) on a square pedestal and spread footing, in the Ramps & Stairs tab.

- Placed by station and distance behind the platform back edge.
- **Grade** as entered, or the lowest grade entered at the piers either side, or the default grade.
- Top of pedestal = grade + reveal (default 8"); bottom of base plate = top of pedestal + grout (platform grout pad unless set); top of base plate = BOBP + plate (default 1"); bottom of footing = grade − depth below grade (default 3'-6"), or set directly; top of footing = BOF + thickness.
- Defaults follow SP-207: 1'-6" pedestal, 4'-6" × 4'-6" × 1'-6" footing, HSS 8×8 column.
- Outputs: a table (station, offset, coordinates, grade, top of pedestal, BOBP, top of base plate, TOF, BOF, pedestal height, cover, checks), a true-scale elevation and plan of the selected foundation with full dimensions and elevations, the footing/pedestal/column in the Ramps & Stairs plan, and footing, pedestal, grout, plate and column in 3D.

### Sheets and DXF
A **Sheets** tab draws 11 × 17 sheets of the ramps, stairs, walls and column foundations, each at a standard scale inside a border and title block (project, project no., designed / checked by and dates, scale, sheet n of N):

- a plan of each ramp and stair along the track: platform edges and piers, footprints with spot elevations and run slopes, walls and dashed footings, column pads and columns, nearby spread-footing columns, dimension chains of every wall face and run end, overall dimensions, the tie-in station and north;
- the developed elevation of every wall (two to a sheet when they still fit at 1/4" = 1'-0"): top of wall at every end, break and pad, slopes, curb, intersecting walls dashed, grout / base plate / column on each pad with BOBP, vertical chains at each end with TOW, grade, TOF and BOF, and horizontal chains of every stretch, the footing toes, the wall and the footing;
- each column foundation on a spread footing: elevation and plan, dimensioned, with every elevation;
- the wall schedule, column schedule (top of concrete, grout, BOBP, plate, top of plate, BOF) and general notes.

**Download DXF** writes one AutoCAD R12 file (model space, full size in inches, the sheets side by side; plot each at the scale in its title block). Layers S-CONC, S-CONC-FTG, S-GRADE, S-WALK, S-XWALL, S-STEEL, S-DIMS, S-TEXT, S-ELEV, S-CNTR, S-PLAT, S-GRID, S-TBLK, S-BRDR; dimensions are DIMENSION entities. **Print sheet / Print all** print each sheet on its own 11 × 17 landscape page at its plot scale. The writer, preview and printing are the All-Structural repo's `shared/dxf.js`, copied into the page.

### 3D details panel
Clicking a component in the 3D view (press and release without dragging) opens a panel inside the app with everything about it: a pier (every elevation, footing, pedestal, beam seats and its section), a beam (section and seat elevations at every support), the platform, a ramp or stair (top, bottom, low grade, segments, profile), a wall (every point, dimensioned elevation), a column on a wall pad, or a column on a spread footing (elevations and the foundation drawing). The picked component is highlighted; **Go to inputs** opens where it is edited; Esc or × closes it.

### Heat map
With *Ramps & stairs* ticked (off by default; elevation only), ramp runs, landings and stair treads are drawn behind the platform on the same colour scale, with walls in grey and contours across the runs. The frame widens to fit them; the contour interval coarsens in whole multiples when the range gets large, and the platform high/low and stats stay platform-only. Untick it to get the full colour range back on the platform.

The platform surface unrolled along the track (station across, distance back from the edge down), coloured by one of:

- **Top of platform elevation** — sequential blue, light = low, dark = high, with contours at a set interval (every fifth labelled). Good for spotting drainage low points and the crest or sag of a vertical curve.
- **Height above rail − target** — diverging around zero with a neutral midpoint; shows where an independent platform profile departs from the design height above the rail.
- **Running slope** — sequential, against the running slope limit.

Support and beam lines overlay the surface; high and low points are marked and reported from a fine scan that includes both platform edges. Hover any cell for station, offset, elevation, deviation and running slope. The view is not to scale across the platform.

### Calculations tab
Pick a support; every elevation is derived as **symbol = formula = substituted values = result**, typeset with KaTeX (CDN; plain-text fallback offline). Sections: track (profile incl. vertical curve equation, cant, rails), horizontal (radius, curve setback, edge coordinates), platform surface, every beam, cap/wall and every bearing, micropiles, pedestal, pedestal pile cap or spread footing. Copy as text and Print.

---

## Saving and defaults

- **Autosave** to browser storage.
- **Library** of named platforms (Save / Save As / Open / Delete), stored in the browser.
- **Export / Import JSON** for a portable copy — includes notes.
- **Set as Default** makes the current inputs the starting point for *New* and for a fresh browser session; *Clear saved default* reverts to the shipped demo.

Browser storage is per-browser and per-machine. Use Export JSON to move work between machines or share it.

---

## Conventions

| Item | Convention |
|---|---|
| Units | Decimal feet internally; feet-inches accepted (`5'-7"`); bearing, panel, grout and plate thicknesses in inches |
| Left / right | Looking up-station |
| Cant | Inches, positive = right rail higher |
| Curve direction | Right/Left selector; radius positive |
| Degree of curve | Chord definition by default |
| Offsets back from the edge | Measured from the platform edge face, normal to the track |
| Supports | Radial (normal to the track) unless skew is unlocked |

---

## Known limitations

- Car body roll from superelevation is not included in the gap checks.
- The 3D view uses painter's algorithm; genuinely interpenetrating geometry may show face-sorting artefacts.
- A spread footing can extend past the platform edge toward the track; it is not checked against the track clearance envelope.
- Micropile tip elevations are intentionally not modelled.
- KaTeX in the Calculations tab needs an internet connection; content still appears as plain text offline.
