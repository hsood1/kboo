# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A personal collection of web gifts for Kboo — self-contained static HTML pages with no build step, no dependencies, and no backend. Each page is a single `.html` file with inline `<style>` (and `<script>` only where noted); you edit the file and reload the browser. There is nothing to install, compile, or bundle.

**Structure — a hub at the root, one folder per page:**

```
index.html                     ← landing hub: cards linking to every page (entry point)
morning/morning.html           ← animated sky, live weather, daily quote, clock, April-1st "Sagan" popup
astrology/astrology.html       ← daily horoscope + Capricorn/Virgo compatibility
nyc-food/nyc-food.html         ← NYC pizza-crawl scrapbook (print-friendly, JS-free)
car-search/car-search.html     ← San Diego hybrid car-buying guide
```

Folder and file names are **hyphen-case** (`nyc-food`, not `nyc_food`). Each page folder may also hold a `*.md` design brief (e.g. `nyc-food/NYC_FOOD.md`, `car-search/CAR_SEARCH.md`) — read it before changing that page; it documents content, intentional decisions, and known issues. Note the briefs predate this multi-page layout and still refer to the page as a standalone `index.html` at a repo root — treat the folder-per-page structure here as the source of truth.

**Navigation:**
- The hub (`index.html`) links out to all four pages.
- `morning` and `astrology` share a toggle pill at the top that switches between just those two, plus a `🏠` link back to the hub. Keep all three pills present and their relative paths (`../`) correct when editing.
- `nyc-food` and `car-search` each have a fixed `🏠 Home` link (inline-styled `<a href="../index.html">`) — they are reached only from the hub, not in the toggle.

## Running / testing

Open a file directly in a browser, or serve the folder to exercise geolocation/`fetch` (some browsers restrict APIs on `file://`):

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

There is no test suite, linter, or CI. Verification is manual: load the page and watch the animation/console.

## Deployment

The repo (`github.com/hsood1/kboo`) is published as a static site — committing to the default branch updates the live page. There is no separate build or deploy command.

## Architecture notes

**External APIs (all keyless, CORS-open, called client-side):**
- Weather: `api.open-meteo.com/v1/forecast` — WMO weather codes are decoded by `decodeWMO()`; `getWeatherLayers()` / `getSuggestion()` map a code to visual layers and copy.
- Location: `navigator.geolocation` first, falling back to `ipwho.is` (`ipFallback()`); `reverseGeocode()` turns coords into a city name. `LOCATIONS` holds hardcoded presets (Poway / Schaumburg) used by `switchLocation()`.
- Horoscope: `aztro.sameerkumar.website` via `fetchFromAztro()`, with a fully local fallback (`generateHoroscope()` + `seededRand()`) when the API fails. `updateSourceBadge()` shows whether the reading is live or generated.

When an external API breaks, the pattern here is **degrade gracefully to a self-generated result**, not to error out — preserve that fallback behavior rather than removing it.

**Visual state is driven by `data-*` attributes on `<body>`:** `data-sky` (sunny / partly-cloudy / overcast / fog / windy / night) and `data-precip` (rain / snow / storm) are independent layers — both can be set at once, and the CSS in `morning.html` keys off them. Change weather visuals by setting these attributes, not by editing element styles directly.

**Determinism for daily content:** the daily quote and horoscopes are chosen by day-of-year (not random per load) so the page shows the same thing all day. See the quote-selection logic and `seededRand()`. Recent history shows repeated regressions here (random vs. seeded, timezone/date resolution) — when touching date logic, confirm it stays stable across reloads within a day and resolves the date in the correct timezone.

**The Sagan popup** (`morning/morning.html`, gated by `SAGAN_ALWAYS_SHOW` and an April-1st check) is an intentional Easter egg. Leave `SAGAN_ALWAYS_SHOW = false` in committed code — it is only flipped temporarily for local debugging.

**`nyc-food` and `car-search` are JavaScript-free by design** — their interactivity (His/Hers and bagel toggles on nyc-food; tab switching, vent toggle, and card expand on car-search) is built with the **CSS checkbox/radio hack**: hidden `<input>`s near the top of `<body>` drive visibility through `~` sibling selectors. The hidden inputs must stay ahead of the content they control, or the selectors silently stop working. Don't reintroduce JS rendering on nyc-food (it's shared as a raw file / printed). See each page's `*.md` brief for the full rationale and known issues (e.g. car-search's card-expand sibling-selector limitation).
