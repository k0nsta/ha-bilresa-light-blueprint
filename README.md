# IKEA BILRESA scroll wheel — light blueprint (Home Assistant)

Flexible, **light-only** Home Assistant blueprint for the IKEA **BILRESA scroll wheel**
(Matter), with **smooth speed-proportional dimming**, **night mode**, and **colour temp /
RGB** control.

[![Open your Home Assistant instance and open a blueprint.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fk0nsta%2Fha-bilresa-light-blueprint%2Fblob%2Fmain%2Fblueprints%2Fbilresa_light.yaml)

## Setup flow

1. **Pick the light(s)** — one, a group, an area, or several.
2. **Pick the 3 wheel events** for ONE group (see groups below).
3. **Choose what the wheel does** — Brightness / Colour temperature / RGB cycle.
4. **Assign each press an action** — single / double / **triple** / long-press → Toggle · On ·
   Off · On-with-preset · Activate a scene · Next colour · Previous colour · Nothing.
   (Set single-click to *Do nothing* if you want only deliberate double/triple presses.)
5. **(Recommended) add a brightness helper** — see below; makes dimming smooth on Matter/Thread.
6. Optional: **preset** (brightness + colour), **colour ranges**, **dimming feel**,
   **transitions**, **night mode**.

## Smooth dimming — the brightness helper

Relative brightness stepping (`brightness_step_pct`) is computed from Home Assistant's
*last-known* brightness. On Matter/Thread bulbs each command takes ~250 ms to report back, so
a fast spin fires steps faster than the bulb updates → they read stale values and
**overshoot or cancel out** (laggy, unpredictable).

Fix: create an **`input_number`** helper (min 1, max 100, step 1) and select it in
**7 · Dimming feel → Brightness helper**. The wheel then bumps the helper **instantly**
(local, ~1 ms, never stale) and sets the bulb to that **absolute** value. Bursts collapse to
the correct final brightness — predictable, no compounding, latency drops to a single
round-trip. Use **one helper per light/group**. Leave it empty to fall back to relative
stepping.

## The 3 groups (and how to use them)

The under-LED button cycles **3 product groups**, so the wheel exposes
**9 button-endpoints = 3 groups × (scroll-up, scroll-down, press)**:

| Group | Scroll up | Scroll down | Press |
|------:|-----------|-------------|-------|
| 1 | `Button (1)` | `Button (2)` | `Button (3)` |
| 2 | `Button (4)` | `Button (5)` | `Button (6)` |
| 3 | `Button (7)` | `Button (8)` | `Button (9)` |

**To control 3 different lights, create 3 automations from this blueprint — one per group.**
Each is fully independent (its own light, actions, colours, night mode). That's more flexible
than one giant config, and you only fill in the groups you actually use.

> Re-paired wheels can leave a **duplicate "orphan" node** — a second set of `Button (1…9)`
> that's `unavailable`. Remove that stale device from the Matter integration so the entity
> picker isn't confusing.

## What makes dimming smooth

Each scroll fires `multi_press_N` (N = notches). Brightness/temp changes by **`step × N`**
(capped) — slow notch nudges, fast spin sweeps. **Transitions default OFF** because many
Matter bulbs *freeze for ~10s* on fast scrolls with transitions on; `mode: queued, max: 4`
also stops bursts from piling up.

## Options

- **Wheel does:** Brightness · Colour temperature (warm↔cool) · RGB cycle.
- **Button actions** (single/double/long): Toggle · On · Off · On-with-preset ·
  **Activate a scene** (e.g. a "Watch TV" scene — pick a scene per button) · Next/Previous
  colour · Nothing.
- **Preset:** brightness + colour (keep / colour-temp / RGB) for "On-with-preset".
- **Colour:** Kelvin range/step for temp, and an editable RGB cycle list.
- **Night mode:** enable + **time schedule** (start/end) + optional override entity; night
  **brightness** + night **colour (temp _or_ RGB)**. During night mode, turn-on actions use
  the night settings.

> Colour next/previous reads the light's current state, so for the most reliable cycling
> point at specific light entities rather than only an area.

## License

MIT.
