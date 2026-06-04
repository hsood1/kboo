# Car Guide — Design Brief

## What this is

A single-file car shopping guide (`car-search.html`) for Harry's girlfriend (Khusbu), who is moving to San Diego and buying/leasing her first car (~September 2026). Hosted on GitHub Pages so she can open it on her phone. Reached from the hub (`../index.html`) via a fixed `🏠` home button (top-right).

## Architecture — now JavaScript-driven

> **Note:** This page was rewritten from the original CSS-only (checkbox/radio hack) version into a JS-driven app. JS is fine here — the page lives on GitHub Pages (HTTPS), not `file://`. The old "zero-JavaScript" constraint no longer applies to this page. (nyc-food is still JS-free by design; car-search is not.)

### Single source of truth
All car data lives in one `CARS` array in the `<script>` at the bottom of `<body>`. Each entry holds pricing (`newLow/High`, `usedLow/High`, `leaseLow/High`, `insLow/High`), `style`, `power`, `budgetFit`, seat status (`heated`, `cooledStatus`), an editorial `rank` (1–10) + `rankNote`, a `tag`/`tagLabel`, optional `caveat`, and `features[]` / `notes[]`. Both the Cards page and the Master table render from this array — **edit a car once, both views update.**

### Pages (6 tabs) — built for a non-technical phone user
`🏁 Start here` **(default)** · `🚗 The cars` · `📊 Compare all` · `🛡️ Insurance` · `📋 Leasing 101` · `💡 Tips`. `showPage(id, btn)` toggles `.page.active`; `showPageById(id)` navigates by name (used by Start-page buttons). The hero header sits above the sticky nav on every tab.

- **Start here** — the friendly landing. `renderTopPicks()` renders the top 3 cars by score as tappable mini-cards (`goToCar(id)` jumps to the Cars tab and flashes that card); plus plain-English lease-vs-buy cards, an in-budget callout, and 3 next-step cards. Set as default so she never lands on a spreadsheet.
- **The cars** — the primary browse view; `renderCards()`.
- **Compare all** — one sortable table (the old "Master" and "Compare" tables were merged — two spreadsheets confused the audience). `renderMaster()` + `sortTable()`. **Score (★) is the first column** and the default sort (desc). The 5-Year Snapshot lives at its bottom.
- Insurance / Leasing 101 / Tips are static HTML.

### Filters — collapsed by default
Four AND-combined rows (Mode · Style · Power · Budget) in `getFiltered()`. They're **hidden behind a `🔍 Filter cars` toggle** (`toggleFilters()` / `applyFilterBarVisibility()`) so browsing — not querying — is the default. The toggle shows an active-count badge and a **Clear** button; `resetFilters()` restores all-to-`all` and is also wired to the "Show all cars" button in every empty state. The toggle row only appears on `filterPages` (`['cars','master']`).

### Mobile UX details
- Pinch-zoom is **enabled** (viewport has no `maximum-scale`) — never trap zoom for this user.
- Horizontal-scroll affordances: a right-edge fade on the tab bar (`.pill-nav-wrap::after`) and on the table (`.hscroll::after`), plus a "↔ Swipe" caption above the table.
- `+Insur/mo` applies to every row including lease; budget fit is the colored left edge of each card/row (no separate column).

### Finance model (computed, not stored)
`loanMo(price, apr)` returns a monthly loan payment assuming **10% down, 72-month term** (new ~6.5% APR, used/CPO ~7.5% APR). On load, an init loop precomputes `newMoLo/Hi` and `usedMoLo/Hi` on each car so cards, the Master table, and sorting all agree. `allIn(c, mode)` returns the **payment + insurance** range for new/used/lease — so **insurance is folded into every monthly figure, leases included**. Cards show "≈$X–Y/mo all-in" under each price; the Master keeps payment and insurance as separate columns. Adjust the constants `DOWN / TERM / NEW_APR / USED_APR` to change assumptions globally.

### Card expand
Each card's "Features & Notes" button calls `toggleCard()` (plain JS, toggles `.open`) — this replaces the old broken sibling-selector approach.

## Car data (estimates, June 2026)

12 cars, ranked editorially:

| Car | Style·Power | Rank | Why |
|-----|-------------|------|-----|
| Honda Accord Hybrid Sport-L | Sedan·Hybrid | 9 | Trusted dealer (Honda Poway), familiar size, cooled seats Sport-L |
| Tesla Model 3 LR | Sedan·EV | 8 | Best lease value, charging solved, cooled standard — **must test drive interface** |
| Toyota RAV4 Hybrid Limited | SUV·Hybrid | 8 | Best CPO buy, reliability/resale, cooled on Limited |
| Toyota RAV4 Prime | SUV·PHEV | 7 | Only PHEV, 42mi EV range, HOV sticker |
| Hyundai Tucson Hybrid Limited | SUV·Hybrid | 7 | Best SUV lease, physical climate dials, dealer uncertain |
| Toyota Camry Hybrid XLE | Sedan·Hybrid | 7 | Strong value, 2025 redesign, dealer TBD |
| RAV4 Hybrid XLE Premium | SUV·Hybrid | 6 | Budget buy, no cooled seats |
| Tucson Hybrid SEL Convenience | SUV·Hybrid | 6 | Cheapest SUV lease, no cooled |
| **Honda Civic Hybrid Sport Touring** | Sedan·Hybrid | 6 | **In budget bought NEW**, trusted Honda Poway dealer, no cooled |
| Hyundai Sonata Hybrid Limited | Sedan·Hybrid | 5 | Good value, dealer concerns — deprioritized |
| RAV4 Hybrid XLE/LE | SUV·Hybrid | 5 | Most affordable, heated may need package |
| **Toyota Corolla Hybrid SE/XLE** | Sedan·Hybrid | 5 | **Cheapest to buy NEW & run** (~50 MPG), compact, no cooled |

**In-budget new options:** most hybrids here are $36K+ new, so Civic Hybrid (~$30–34K) and Corolla Hybrid (~$26–31K) were added as cars genuinely affordable bought new at ~$30K — the honest trade is they're smaller and lack cooled seats.

**Budget fit** drives the card left-border stripe: 🟢 near (≤~$5K over $30K buy, or <$450/mo lease) · 🟡 somewhat over · 🔴 significantly over.

## Buyer context (do not invent beyond this)

- **Wants:** hybrid/EV, heated AND cooled seats ideally, simple non-overwhelming interface, SUV or sedan (sedan now fine — Harry drives an Accord; Khusbu is comfortable sedan-sized).
- **Excludes:** Kia (personal preference).
- **Budget:** ~$30K to buy, or lease. First solo insurance policy (~$200–280/mo to start).
- **Location/timing:** buy & register in California (any SoCal — El Cajon, Temecula, etc.). ~September 2026; end-of-quarter is best for negotiating.
- **Charging:** work has a free charger; home likely will too → Tesla running cost ~$20–30/mo.

### Dealer intelligence (facts only)
- **Honda Poway** — Harry has multiple positive firsthand experiences. Verified.
- **Hyundai (SD area)** — Harry has concerns; no specific incident. Treat as unverified; don't recommend without caveat.
- **Toyota (SD area)** — Mossy Toyota (Kearny Mesa), Toyota of El Cajon commonly cited; unverified by Harry personally.
- **Tesla** — no dealership; single SD service center, sales direct.

## Design system

- **Fonts:** Playfair Display (display/headings), DM Sans (body). 15px base.
- **Colors (CSS vars):** `--cream` page bg, `--charcoal` headers/dark sections, `--accent` warm orange, `--green`/`--blue`/`--red`/`--amber` semantic, `--border`.
- **Budget stripe:** `.car-card.budget-green/amber/red` left border; mirrored on Master rows.

## Deployment
Edit `car-search.html`, commit to `main`, push. GitHub Pages auto-deploys. No build step.
