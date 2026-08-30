# Changelog

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
