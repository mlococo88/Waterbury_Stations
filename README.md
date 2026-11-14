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
- **3D**: painter's-algorithm renderer, no external library. Layers for platform, tactile strip, beams (true wide flanges), cap beams/footings, micropiles, pedestals, canopy columns, track, ground (interpolated from per-support grades, adjustable opacity), and support labels. Station window, vertical exaggeration, pile stub length.

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
