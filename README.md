# IKEA BILRESA scroll wheel — smooth light blueprint (Home Assistant)

A **light-only** Home Assistant blueprint for the IKEA **BILRESA scroll wheel** (Matter),
focused on **smooth, speed-proportional dimming** — the thing most community blueprints
get wrong.

[![Open your Home Assistant instance and open a blueprint.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fk0nsta%2Fha-bilresa-light-blueprint%2Fblob%2Fmain%2Fblueprints%2Fbilresa_light.yaml)

## How the wheel works

Over Matter the BILRESA is a **generic switch with 9 button-endpoints**, exposed as
`event.bilresa_scroll_wheel_button_1…9`. The **"control under the LED"** cycles **3 product
groups**, so the 9 endpoints are **3 groups × (scroll-up, scroll-down, center-press)**:

| Group | Scroll up | Scroll down | Center press |
|------:|-----------|-------------|--------------|
| 1 | `button_1` | `button_2` | `button_3` |
| 2 | `button_4` | `button_5` | `button_6` |
| 3 | `button_7` | `button_8` | `button_9` |

> Your entity IDs may carry a suffix (e.g. `…button_4_2`) if the wheel was re-paired —
> just pick whatever HA shows.

- **Center press** → `multi_press_1` (single), `multi_press_2` (double), `multi_press_3`
  (triple), `long_press` / `long_release` (hold).
- **Scroll** → `multi_press_N`, where **N = notches in the gesture**. A slow notch is
  `multi_press_1`; a fast spin coalesces into `multi_press_2…8`.

## The smoothness trick

Each scroll changes brightness by **`step % × N`** (capped), with a short **transition**.
So a slow notch nudges and a fast spin sweeps — instead of a fixed tiny step per event.

## What it does

| Control | Action |
|---------|--------|
| **Scroll up / down** | Smooth dim up/down (`step × N`, speed-proportional) |
| **Single press** | Toggle — *night-aware* (turns on at night brightness/colour during night mode) |
| **Double press** | Next colour (warmer, or next RGB in the cycle) |
| **Triple press** | Previous colour (cooler, or previous RGB) |
| **Long press** | Off |

## Options

- **Dimming feel** — brightness per notch, max speed multiplier.
- **Transitions** — on/off toggle + time. *Off by default* (transitions freeze many Matter bulbs on fast scrolls).
- **Colour** — mode = **Off / Colour temperature / RGB**.
  - *Colour temperature*: double = warmer, triple = cooler, within a configurable Kelvin range/step.
  - *RGB*: double/triple cycle a configurable list of colours.
- **Night mode** — enable + **time schedule** (start/end), with an optional override entity
  (`input_boolean` / `binary_sensor` / `schedule`). During night mode, turning the light on
  uses a **night brightness** and **night colour temp**. Dimming/colour still work normally.

> Colour reads the light's current state, so point at specific light entities (not just an
> area) for the most reliable next/previous cycling.

## Setup

1. Import via the button above (or Settings → Automations → Blueprints → Import, paste the
   blueprint URL).
2. Create an automation from it and fill:
   - **Scroll-up / Scroll-down / Center-press** event entities for **one group**
     (e.g. Group 2 → `…button_4`, `…button_5`, `…button_6`).
   - **Light / area** to control.
   - **Dimming feel**: brightness per notch, max speed multiplier, transition.
3. Want the wheel to control **3 different lights**? Use the under-LED button to switch
   groups, and create **one automation per group** (groups 1/2/3) pointing at different lights.

## Notes

- If you re-paired the wheel, the **old Matter node may linger** as a duplicate set of
  `unavailable` `button_1…9` entities — remove the stale device from the Matter integration.
- License: MIT.
