# Changelog

## 1.14.0

- All three performance presets now run the same 300 ms step. That is where the two
  beats read best, and a slower step is also less work per second, so the step was
  never what the presets should have been trading away. They still differ in
  `maxsteps` and in painted effects, so a weaker tablet gets a board that settles
  sooner rather than one that flaps faster
- Measured settle times are now 4.3 s at `high`, 3.1 s at `mid` and 2.3 s at `low`,
  up from 3.2 / 2.0 / 1.5. Lower `maxsteps` if that is too long to look at
- The URL parameter table gave "preset" as the default for `ms`, `lite`, `flip` and
  `maxsteps`, which meant looking them up in the source. It now gives the resolved
  values, with a second table for the two that actually differ between presets
- The table also claimed `laps` defaults to `1`; it is and was `0`. The settle-time
  section was written before `maxsteps` existed and quoted times that cannot occur at
  the default cap, because `shorten()` zeroes `laps` whenever the journey does not fit
  in `maxsteps` - which at the default is always. That `laps` is inert unless
  `maxsteps=0` is now stated rather than left to be discovered

## 1.13.2

- Fixed the real cause of the flap reading as a single event: only the first flap of
  a run ever animated. `settle()` strips both flip classes at the start of a step, so
  `flipClass()` could no longer tell from the element which one had just run and put
  the same one back every time. Nothing in between touched style, so the browser
  resolved one unchanged value and never restarted the animation - every step after
  the first snapped both halves over at once. The cell now remembers which class it
  used
- `ms` was capped at 300, so the slow-motion values suggested for checking the flap
  were silently clamped and the test could not work. The cap is now 2000, in the page
  and in the `flightwall_ms` helper alike
- The flap speed documentation still described a default of 76 and warned against
  going above 120, which is the opposite of what the presets have done since 1.13.0

## 1.13.1

- Fixed the flap reading as a vertical squash rather than a fall. `perspective` had
  been moved from the cell to the row as a supposed saving, which put its origin at
  the centre of the row: cells near the middle then got almost no perspective at all

## 1.13.0

- Presets rebalanced toward fewer, slower flaps. The two-beat flap was already in
  place but invisible at 76 ms per step, because each beat lasted under 40 ms. All
  three presets now sit above the threshold where the mechanism reads, and all three
  do less work than the old `high` did

## 1.12.0

- The two leaves of a flap now fall in sequence rather than together: the upper leaf
  drops over the first half of the step, the lower one swings up over the second.
  Running both at once read as a fold rather than a fall
- The logo flap now turns through a drum of carriers (Lufthansa, British Airways,
  Condor, Delta, plus the previous and the current airline) instead of flipping once.
  It is now an ordinary cell whose set holds image URLs, so `maxsteps` and `laps`
  apply to it as well
- Helpers for every remaining parameter: `flightwall_maxsteps`, `flightwall_ms`,
  `flightwall_flip`, `flightwall_intro`. The numeric ones use `-1` for "leave it to
  the preset", so `perf` still means something

## 1.11.0

- New `maxsteps` cap on how many flaps a cell makes. Measured on a headless run, this
  takes a refresh from about 32,000 text writes down to 1,900 at `perf=low`
- All three presets keep the 3D flip. Disabling it in `low` turned out not to help,
  because the flip was never the bottleneck; the number of steps was
- Removed `will-change: transform` from the leaves. With four halves per cell it was
  promoting 256 elements to their own compositing layers, which costs more than it
  saves on weak hardware

## 1.10.1

- Fixed cells freezing with the old character on the top half and the new one on
  the bottom. Introduced with the requestAnimationFrame rewrite: dropping the flip
  class is not enough, the cell also has to go back to `idle`, or the leaves stay
  visible with no animation covering them

## 1.10.0

- Extra passes are now a number helper, `input_number.flightwall_laps`, alongside the
  performance dropdown. Because the automation always sends it, it overrides whatever
  the performance preset would have picked

- Animation budget is now a dropdown helper, `input_select.flightwall_perf`, passed
  through to the board by the automation. No YAML editing to change it, and it can
  live on a dashboard next to the style picker
- The board falls back to `high` if the helper is missing or unset

## 1.9.0

- All cells are now driven by a single `requestAnimationFrame` loop instead of one
  `setTimeout` chain per cell, which at 128 cells meant over a thousand timer
  callbacks a second, each with its own style recalculation
- New `perf` preset, `high` | `mid` | `low`. `low` also disables the 3D flip: the
  characters still cycle, but nothing rotates, removing the leaf layers and 3D
  transforms entirely
- Individual parameters now override the preset rather than the other way round

## 1.8.0

- Cells now take one extra full pass through the character set before settling, so
  the whole board moves on every update rather than only the cells whose character
  changed. New `laps` parameter, `0` restores the previous shortest-path behaviour

## 1.7.0

- Much faster on low-powered tablets. The flip no longer forces a synchronous reflow
  on every step, which previously meant several thousand forced layouts per update
- Shading is now a static difference between leaf faces instead of an animated
  `filter: brightness()`, which forced a repaint every frame
- Cells use `contain: layout paint style`, and `perspective` moved from every cell to
  the row
- New `lite=1` parameter trimming shadows, radii and glows

## 1.6.0

- Carrier logo flap in the leftmost cell of the FLIGHT row, turning once per update
  with the same two-leaf mechanism as the character cells
- ESTIMATED row picked out in amber, as on real departure boards
- FLIGHT value now truncates at 14 characters, since two cells go to the logo flap
  and its gap

## 1.5.0

- Split-flap board now opens by restoring the previous flight, blinking a green
  FLIGHT INCOMING banner, and only then flapping to the new values. The flaps
  therefore start from the previous letters instead of from blank
- Previous flight is kept in the page's own `localStorage`, so no Home Assistant
  helper and no automation change is needed
- New URL parameters: `intro`, `hold`, `blink`

## 1.4.1

- Fixed intermittent "stopped for unknown reason" aborts. The state trigger had no
  `to:`, so it also fired on attribute changes, and altitude and distance change on
  every poll. Combined with `mode: restart`, a poll landing mid-run killed that run.
  The trigger now uses `to: ~` and the mode is `single`

## 1.4.0

- Screenshots of both styles in the README

## 1.3.1

- Close button no longer shows a grey rounded box. The button card paints its own
  card background and a ripple layer, neither of which the earlier transparent
  background rule reached

## 1.3.0

- Split-flap row order is now FLIGHT, AIRCRAFT, AIRLINE, TO, ESTIMATED, ALTITUDE,
  AIRSPEED, PROGRESS
- Aircraft type replaces distance. The readable
  model is used when it fits in 16 characters, otherwise the ICAO code, so it never
  truncates mid-word
- AIRLINE no longer falls back to the aircraft type, since that has its own row
- The screen wake now fires after the popup is sent rather than before, so the page
  is already loading when the display comes on and the flap animation is not missed

## 1.2.0

- Split-flap board gains printed row labels: FLIGHT, AIRLINE, TO, ESTIMATED,
  ALTITUDE, AIRSPEED, AIRCRAFT, PROGRESS
- One value per row instead of packed lines, so the board is 16 characters wide
  rather than 22
- ESTIMATED shows the arrival clock time alongside the countdown
- Flap speed halved (`ms` default 38 to 76) and the column sweep slowed to match

## 1.1.0

- Split-flap display style, selectable via `input_select.flightwall_style`
- Standalone `www/splitflap.html` board with per-character mechanical flip animation,
  alphabet cycling, and a staggered column sweep
- Fixed: aircraft with no filed flight plan showed a fully complete progress bar in
  both styles, because the elapsed/total division clamps its denominator to 1

## 1.0.0

Initial release.

- Elevation-angle aircraft selection
- Dot-matrix full-screen display with airline logo, route, type, status and telemetry
- Flight progress bar
- One popup per aircraft per zone entry
- Full-screen popup via browser-mod, with a close button
- Optional Fully Kiosk screen wake
