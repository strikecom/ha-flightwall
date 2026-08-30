# Customisation

## Behaviour

### Quiet hours

In the automation, the `condition: time` block. The "clear" branch deliberately has
no time condition, so the popup still closes at night.

```yaml
  - condition: time
    after: "07:00:00"
    before: "22:00:00"
```

Delete the block for 24-hour operation.

### How long the popup stays

Two values, and they are independent:

- `timeout: 60000` in the automation (milliseconds)
- nothing else — the progress bar shows flight progress, not a countdown

### How long the display stays active after the last aircraft

`delay_off` on `binary_sensor.flightwall_inbound` in the package file. Two minutes by
default. This is the single most important value for the feel of the thing: too short
and the screen flickers between polls, too long and it stays on after the sky is
empty. Near a busy airport, raise it. In quiet airspace, lower it to 60 seconds.

### Showing every aircraft, including repeats

Remove this condition from the automation:

```yaml
  - condition: template
    value_template: >
      {{ states('sensor.flightwall_flight') != states('input_text.flightwall_shown') }}
```

Expect the same aircraft to reappear as the elevation-angle ranking shifts between
two nearby planes.

### Sorting by distance instead of elevation angle

Not recommended, but if you want it, replace the `namespace` loop in the template
sensor with:

```jinja
{% set f = (state_attr('sensor.flightradar24_flights_in_area','flights') or [])
   | selectattr('distance','defined')
   | sort(attribute='distance') | first | default(none) %}
```

You will get cruising traffic at 38,000 ft rather than the aircraft overhead.

### Minimum altitude

The template filters `altitude > 500` to exclude aircraft on the ground. If you live
inside an airport's radius, taxiing aircraft would otherwise permanently win the
ranking. Raise it if you only care about aircraft at cruise.

## Appearance

All in the `card_mod` block.

### Dot-matrix mask

```css
ha-card::after {
  background-image: radial-gradient(circle at center,
    rgba(0,0,0,0) 0 50%, #000 51% 100%);
  background-size: 4px 4px;
}
```

- `background-size` is the dot pitch. Smaller means finer dots and higher apparent
  resolution, but thin glyph strokes start disappearing. 4–7 px is the useful range.
- The two percentages are the hole size. `50%`/`51%` is a hard edge — keep the gap
  at one percentage point. A wider gap (`44%`/`50%`) produces soft rings that read as
  blur, which is the most common way this effect goes wrong.
- Delete the whole `::after` block for plain text with no mask.

### Stroke weight

Press Start 2P has no bold cut, so the browser would synthesise one and shift the
glyphs. Instead the text is thickened with three offset copies:

```css
text-shadow:
  0.04em 0 0 currentColor,
  0 0.04em 0 currentColor,
  0.04em 0.04em 0 currentColor;
```

Raise `0.04em` for heavier strokes, lower it for finer. This matters more than the
mask settings for legibility at a distance.

### Font sizes

Every size uses `min(Xvw, Yvh)` — the smaller of a percentage of width or height.
This means text can never overflow either edge regardless of screen aspect ratio or
orientation.

| Element | Content | Default |
|---|---|---|
| `h1` | Route | `min(12vw, 18vh)` |
| `h2` | Callsign and airline | `min(4.2vw, 5.8vh)` |
| `h3` | Aircraft type | `min(4vw, 5.6vh)` |
| `p` | Status and telemetry lines | `min(3.3vw, 4.7vh)` |
| `p:last-of-type` | Progress bar | `min(2.8vw, 4vh)` |

The constraint is the longest status line ("DEPARTED DUSSELDORF 19M AGO", 27
characters), not the route. If you scale up and it wraps, shrink the logo column
first.

### Logo size

```css
grid-template-columns: min(34vw, 48vh) minmax(0, auto);
```

and

```css
p:first-of-type img { width: min(32vw, 46vh); }
```

Keep the column slightly wider than the image. The column has a **fixed** width
deliberately: if it were `auto`, a missing logo would collapse it and shift the whole
layout sideways.

### Colours

| Element | Value |
|---|---|
| Route, status lines | `#ffffff` |
| Callsign and airline | `#9fb8e8` |
| Aircraft type | `#7ea6ff` |
| Progress bar | `#35ff7a` |
| Close button | `rgba(255,255,255,.28)` |

The progress bar uses `-webkit-text-fill-color` alongside `color` because HA theme
rules can override plain `color` at equal specificity.

### Monochrome logos

Replace the image filter with:

```css
filter: grayscale(1) brightness(1.9) contrast(1.5);
```

More authentic to a real single-colour LED panel, and it rescues logos that are white
on transparent and would otherwise vanish against black.

### Progress bar characters

```jinja
{{ '█' * prog }}{{ '░' * (32 - prog) }}
```

Both are full-cell block characters in the same colour, which is why they render
identically. The bar deliberately does **not** use `<strong>` or a second colour:
Press Start 2P has no bold cut and no block glyphs, so either would trigger a font
fallback and the two halves would no longer line up. This is also why the bar alone
uses `font-family: monospace` rather than the pixel font.

Change `32` in both places to adjust the length.

## Tablet setup (Fully Kiosk)

1. Install Fully Kiosk Browser plus the Plus licence.
2. Enable **Remote Administration** and set a password.
3. Set the screensaver timeout to something longer than your expected popup duration
   — otherwise the tablet sleeps mid-display.
4. Put the tablet's IP and the password into the `rest_command` in the package file.

If you do not use Fully, delete the `rest_command` from the package and remove this
line from the automation:

```yaml
  - action: rest_command.tablet_stop_screensaver
```

Two systems fighting over the screen is the usual failure mode here. Let one own it.

## Split-flap style

The board is `www/splitflap.html`, served from `/local/flightwall/splitflap.html` and
embedded in an iframe. Open it in a browser with no URL parameters to get a demo
board — that is the fastest way to iterate, since you are not waiting for aircraft.

### URL parameters

| Parameter | Meaning | Default |
|---|---|---|
| `r1`–`r7` | Row values, in label order | demo text |
| `l1`–`l8` | Label overrides | see below |
| `p` | Progress, `0` to `cols` | `10` on the demo board, else `0` |
| `cols` | Board width in characters | `16` |
| `ms` | Milliseconds per flap step, `12`-`2000` | `300` |
| `laps` | Extra full passes before a cell settles, `0`–`3` | `0` |
| `logo` | Airline IATA code, or a full image URL | `LH` on the demo board, else none |
| `perf` | `high`, `mid` or `low` preset for weaker tablets | `high` |
| `lite` | `1` trims painted effects | `0` |
| `flip` | `0` swaps characters without the 3D flip | `1` |
| `maxsteps` | Cap on flaps per cell, `0` uncapped | `14` |
| `intro` | `0` disables the incoming-flight sequence | `1` |
| `hold` | Milliseconds the previous flight stays visible | `900` |
| `blink` | Milliseconds the banner blinks | `2200` |

The defaults above are what `perf=high` resolves to, since that is the default preset.
Only two of them change with `perf`, and `ms`, `laps` and `flip` are the same in all
three:

| `perf` | `lite` | `maxsteps` |
|---|---|---|
| `high` | `0` | `14` |
| `mid` | `1` | `8` |
| `low` | `1` | `5` |

Default labels: `FLIGHT`, `AIRCRAFT`, `AIRLINE`, `TO`, `ESTIMATED`, `ALTITUDE`,
`AIRSPEED`, `PROGRESS`.

The automation builds this URL with Jinja and `urlencode`. To change what a row says,
edit the `r1`–`r7` assignments in the iframe card's `url:` template; to change what it
is called, pass `l1`–`l8`. Labels are printed on the frame, not set in flaps, so
changing them costs no animation time.

### Board width

Passing `cols` alone is not enough — three things must agree:

1. `cols` in the URL
2. the `[:16]` truncations in the automation's `url:` template
3. the `* 16` in the progress calculation

The page clamps `cols` to 8–48. Everything scales off `--cell-h`, which is
`min(9.6vh, 94vw / 13.4)` — eight rows tall, and wide enough for the label column
plus every cell. Raising `cols` much past 20 will overflow horizontally; adjust the
`13.4` divisor to compensate.

### Flap speed and feel

`ms` is the dwell per step, and comes from the `perf` preset unless you pass it. A
step is two beats, so it also sets how long each one lasts: below roughly 120 the
fall and the rise blur into a single event and the board stops reading as mechanical.
Above about 400 nothing breaks, but it is slow enough that it is only really useful
for watching the mechanism - `?ms=1400` makes the two beats unmistakable. The page
clamps it to 12-2000.

Total settle time is driven by the character set length, because a real board cycles
through its physical sequence in order. `MAIN` has 46 entries, so a worst-case cell
does 45 flips. Shortening `MAIN` speeds everything up at the cost of dropping
characters — anything not in the set renders as a blank cell.

Column stagger is in `start()`:

```js
setTimeout(function () { step(c); }, c.col * 52 + Math.random() * 90);
```

`c.col * 52` is the left-to-right sweep; the random term keeps columns from moving in
lockstep. Set the random term to `0` for a rigid mechanical sweep, or drop the `col`
term for all columns starting at once.

### Carrier logo flap

The leftmost cell of the FLIGHT row is a logo flap rather than a character cell, as
carrier logos were on real Solari boards. It uses the same two-leaf mechanism and
turns once per update, from the previous carrier to the new one.

Pass `logo` as an IATA code and the page builds a Kiwi CDN URL from it, or pass a
full URL or a `/local/...` path to use your own asset. With no `logo` the cell stays
blank and nothing turns.

This is the one place where the logo resolution finally works out: a cell is about
100 px tall on a 1080p tablet and the CDN images are 128 px, so for once the source
is larger than the display size.

The FLIGHT row is indented by two cells to clear the flap, which is why its value is
truncated at 14 characters rather than 16. `LOGO_ROW` and `LOGO_PAD` at the top of
the script control this; setting `LOGO_ROW` to `2` moves the flap to the AIRLINE row.

### Incoming-flight sequence

On load the board runs three phases:

1. The previous flight is restored instantly from `localStorage`, with no animation
2. A green `FLIGHT INCOMING` banner blinks over a dimmed board
3. The flaps run to the new values

Because phase 1 puts the old characters in the cells, the flaps start from those
rather than from blank, which is how a real board behaves. It also means a cell whose
character is unchanged simply does not move.

The whole sequence adds `hold + blink` before the flaps begin, about 3.1 seconds at
the defaults, on top of roughly 4.3 seconds of flapping. Set `intro=0` to go straight
to the flaps.

Nothing is stored on the Home Assistant side: the page writes the current values to
`localStorage` under `flightwall.previous` on every load and reads them back on the
next one. On the very first load, or in a browser with storage disabled, there is no
previous flight and the sequence is skipped automatically.

### Settle time

`maxsteps` is what decides this in practice, and at its default of `14` it settles
the question on its own: the worst cell walks 14 steps, so the board is at rest after
about 4.3 seconds. `mid` and `low` cap it lower and finish sooner.

`laps` only comes into play once the cap is lifted. `shorten()` compares the whole
journey against `maxsteps` and zeroes `laps` whenever it does not fit, which at the
default cap is always. **`laps` therefore does nothing unless you also pass
`maxsteps=0`.**

With the cap off, each lap adds one full pass through the 46-character set:

| `laps` | Worst-case steps | At `ms=300` | At `ms=200` |
|---|---|---|---|
| `0` | 45 | 13.5 s | 9.0 s |
| `1` | 91 | 27.3 s | 18.2 s |
| `2` | 137 | 41.1 s | 27.4 s |

`laps=0`, the default, takes the shortest path: each cell stops the first time it
reaches its character, and a cell whose character has not changed never moves. Sparse
and true to a real board, but visually thin when only a few values differ.

`laps=1` sends every cell all the way round once more before it settles, so the whole
board moves on every update. Uncapped that is close to half a minute of flapping, so
it is worth pairing with a lower `ms`.

If the board takes too long to settle, lower `maxsteps` rather than `ms` — fewer
flips at 300 ms reads better than more flips at a pace where the two beats blur.

### Performance on older tablets

### Helpers

Everything is switchable from a dashboard. The package creates:

| Helper | Effect |
|---|---|
| `input_select.flightwall_style` | dot-matrix or split-flap |
| `input_select.flightwall_perf` | `high`, `mid` or `low` preset |
| `input_number.flightwall_laps` | extra passes, `-1` leaves it to the preset. Inert unless `maxsteps` is `0` |
| `input_number.flightwall_maxsteps` | flap cap, `-1` leaves it to the preset |
| `input_number.flightwall_ms` | step duration, `-1` leaves it to the preset |
| `input_boolean.flightwall_flip` | the 3D flip itself |
| `input_boolean.flightwall_intro` | the opening sequence |

The numeric helpers use `-1` for "leave it to the preset", and the automation only
puts a parameter in the URL when it is set. Without that, every helper would override
the preset all the time and `perf` would stop meaning anything.

Set this from the dashboard rather than the URL: the helpers are passed straight
through to the board by the automation, so you can change it on the
tablet itself and trigger the next popup to see the result.

For quicker iteration, open the board directly with the parameter appended, which
needs no aircraft and no automation run:

```
http://your-ha:8123/local/flightwall/splitflap.html?perf=low
```

Start with the `perf` preset and only reach for individual parameters if it is not
enough. Any explicit parameter overrides the preset, so `perf=low&ms=200` is valid.

All three presets run the same 300 ms step, because that is where the mechanism reads
best and a slower step is also less work per second. What they trade away is
`maxsteps` and painted effects, so a weaker tablet gets a board that settles sooner
rather than one that flaps faster.

| `perf` | `ms` | `maxsteps` | Beat 1 | Beat 2 | Text writes | Settle |
|---|---|---|---|---|---|---|
| `high` | 300 | 14 | 156 ms | 144 ms | ~3,800 | 4.3 s |
| `mid` | 300 | 8 | 156 ms | 144 ms | ~2,500 | 3.1 s |
| `low` | 300 | 5 | 156 ms | 144 ms | ~1,700 | 2.3 s |

Write counts are measured, not estimated, by instrumenting `textContent` while the
board runs headlessly.

**The two parameters do different jobs, and `ms` is the one that decides how
mechanical it looks.**

`ms` is how long a single flap takes. A flap is two beats: the upper leaf falls, then
the lower one swings up. Below roughly 120 ms the two beats blur into one event and
the board stops reading as mechanical, no matter how many flaps it makes. An earlier
version used 76 ms, and the sequential motion was simply invisible.

`maxsteps` is how many flaps a cell makes. It has almost nothing to do with realism
and almost everything to do with load: it is the difference between 128 cells doing
five updates each and doing forty-five.

So realism comes from **fewer, slower** flaps rather than many fast ones, which is
also why the presets got lighter as they got more convincing.

`laps` sends every cell all the way round the set once more, so that even a cell
whose character has not changed moves. It is a stylistic choice rather than a
realistic one, and it multiplies the work: leave it at `-1` unless you want it.

### Suggested starting points

| You want | `ms` | `maxsteps` | `laps` |
|---|---|---|---|
| The default | 300 | 14 | -1 |
| Snappier, still mechanical | 200 | 8 | -1 |
| Whole board churns on every update | 300 | 25 | 1 |
| Weak hardware | 300 | 5 | -1 |

Change one at a time. `maxsteps` first, since it decides how long the board takes to
settle, and leave `ms` alone unless the flaps themselves feel wrong: it decides
whether the mechanism is visible at all, and 300 is where it reads best.

To check the mechanism itself, slow it right down and cut it to a single flap per
cell:

```
...splitflap.html?ms=1200&maxsteps=1&laps=0&intro=0
```

At 1200 ms each beat lasts over half a second, so the upper leaf visibly drops before
the lower one swings up. If the two still look simultaneous there, the problem is the
rendering rather than the speed, and `www/flaptest.html` will tell you which variant
your device does render correctly: it shows four implementations side by side at 1.4
seconds per step.

If it still stutters at `perf=low`: `maxsteps=3`, then `cols=12` (and match the
truncation in the automation), then `flip=0`, then `intro=0`.

Helpers override the preset when they are set, so leave them at `-1` unless you are
deliberately tuning one.

### How a flap works

Each cell holds four layers: static `top` and `bottom` halves showing the current
character, plus two animating leaves.

A step runs in **two beats**, which is what makes it read as a falling flap rather
than a fold. The top leaf, carrying the old character, drops away over the first 52%
of the step. Only then does the bottom leaf, carrying the new character, swing up
over the remaining 48%. The static halves are swapped underneath while each leaf
covers them, so the seam never shows a mismatch.

`perspective` sits on the cell, not the row. It looks like an easy saving to move it
up a level, but its origin then lands at the centre of the row: cells near the middle
get almost no perspective and their rotation reads as a vertical squash rather than a
fall. An earlier version made exactly that mistake, and the two beats became
invisible even at a slow `ms`.

`animation-fill-mode: backwards` on the lower leaf matters: without it the leaf would
sit at its final upright position during its delay, showing the new character before
the old one has finished falling.

The logo flap is an ordinary cell whose set holds image URLs instead of characters,
so it cycles through the stock carriers on its drum exactly as a letter cycles
through the alphabet, and `maxsteps` and `laps` apply to it too.

The `filter: brightness()` in the two keyframes is what sells the effect — the
falling leaf darkens as it rotates away from the light, the rising one brightens.
Remove it and the flip looks like a fold rather than a physical flap.

### Colours

The ESTIMATED row is amber (`--amber`), matching the highlighted estimated-time
column on real German departure boards. Board `#08090b`, flap faces `#1a1c21`, lit faces `#23262c` during a flip, ink
`#f4f3ef` (slightly warm, to read as painted flaps rather than backlit pixels),
labels `#8b9099`, and the progress strip `#35ff7a`. All in `:root` at the top of the
HTML.
