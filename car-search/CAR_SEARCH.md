# Car Guide — Design Brief

## What this is

A single-file car shopping guide (`car-search.html`) for the buyer — moving to San Diego and buying/leasing a first car (~September 2026). Hosted on GitHub Pages so it opens easily on a phone. Reached from the hub (`../index.html`) via a fixed `🏠` home button (top-right).

## Architecture — now JavaScript-driven

> **Note:** This page was rewritten from the original CSS-only (checkbox/radio hack) version into a JS-driven app. JS is fine here — the page lives on GitHub Pages (HTTPS), not `file://`. The old "zero-JavaScript" constraint no longer applies to this page. (nyc-food is still JS-free by design; car-search is not.)

### Single source of truth
All car data lives in one `CARS` array in the `<script>` at the bottom of `<body>`. Each entry holds pricing (`newLow/High`, `usedLow/High`, `leaseLow/High`, `insLow/High`), `style`, `power`, `budgetFit`, seat status (`heated`, `cooledStatus`), an editorial `rank` (1–10) + `rankNote`, a `tag`/`tagLabel`, optional `caveat`, and `features[]` / `notes[]`. Both the Cards page and the Master table render from this array — **edit a car once, both views update.**

### Pages (6 tabs) — built for a non-technical phone user
`🏁 Start here` **(default)** · `🚗 The cars` · `📊 Compare all` · `🛡️ Insurance` · `📋 Leasing 101` · `💡 Tips`. `showPage(id, btn)` toggles `.page.active`; `showPageById(id)` navigates by name (used by Start-page buttons). The hero header sits above the sticky nav on every tab.

- **Start here** — the friendly landing. `renderTopPicks()` renders the top 3 cars by score as tappable mini-cards (`goToCar(id)` jumps to the Cars tab and flashes that card); plus plain-English lease-vs-buy cards, an in-budget callout, and 3 next-step cards. Set as default so the buyer never lands on a spreadsheet.
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

15 cars, **scored with the $30K budget weighted heavily** (cheap in-budget cars rise; $48K+ cars fall even if great):

| Car | Style·Power | Score | Budget | Why |
|-----|-------------|:---:|:---:|-----|
| Honda Accord Hybrid Sport-L | Sedan·Hybrid | 9 | 🟢 | Budget-smart sweet spot: CPO ~$28–33K, Honda Poway, cooled seats |
| Toyota Camry Hybrid XLE | Sedan·Hybrid | 8 | 🟢 | Cooled seats + CPO in budget + Toyota resale (dealer unverified) |
| Honda Civic Hybrid Sport Touring | Sedan·Hybrid | 8 | 🟢 | In budget bought NEW, trusted Honda Poway dealer |
| Tesla Model 3 LR | Sedan·EV | 7 | 🟡 | Best lease + charging solved, but pricey to buy / high insurance |
| Toyota RAV4 Hybrid XLE/LE | SUV·Hybrid | 7 | 🟢 | Most affordable RAV4, great LE lease |
| Toyota Corolla Hybrid | Sedan·Hybrid | 7 | 🟢 | Cheapest to buy NEW & run (~50 MPG), compact, no cooled |
| **Toyota Prius LE/XLE** | Sedan·Hybrid | 7 | 🟢 | **NEW:** efficiency champ (~57 MPG), in budget, no cooled |
| Toyota RAV4 Hybrid Limited | SUV·Hybrid | 6 | 🟡 | Cooled seats but ~$48–51K new → only fits via CPO |
| RAV4 Hybrid XLE Premium | SUV·Hybrid | 6 | 🟢 | In-budget RAV4 without cooled seats |
| Tucson Hybrid SEL Convenience | SUV·Hybrid | 6 | 🟢 | Cheapest SUV lease, physical dials, dealer uncertain |
| **Ford Escape Hybrid ST-Line** | SUV·Hybrid | 6 | 🟢 | **NEW brand:** in-budget SUV (esp. CPO), no cooled, avg reliability |
| **Hyundai Elantra Hybrid Limited** | Sedan·Hybrid | 6 | 🟢 | **NEW:** cheapest in-budget + **cooled seats**; Hyundai dealer caveat |
| Toyota RAV4 Prime | SUV·PHEV | 5 | 🔴 | Only PHEV (42mi EV, HOV) but most expensive (~$47–52K) |
| Hyundai Tucson Hybrid Limited | SUV·Hybrid | 5 | 🟡 | Great interface but CPO-only budget + dealer uncertain |
| Hyundai Sonata Hybrid Limited | Sedan·Hybrid | 5 | 🟢 | In budget w/ cooled, but Hyundai dealer concerns — deprioritized |

**Three brands/options added** to broaden the in-budget set beyond Toyota/Honda: **Ford Escape Hybrid** (SUV), **Toyota Prius**, **Hyundai Elantra Hybrid** (cheapest route to cooled seats). No Kia (excluded).

**Budget fit** = three tiers driving the card/row left-edge stripe and the Budget filter: `'in'` 🟢 (attainable ~$30K new or via mild CPO) · `'stretch'` 🟡 ($48K-ish cars that only reach budget as CPO) · `'over'` 🔴 (expensive even used). Hand-set per car (editorial, budget-weighted), not auto-computed.

**In-budget filter:** the Budget filter row (All / 🟢 In Budget / 🟡 Stretch / 🔴 Over), plus a always-visible **"🟢 Show in-budget only" chip** on the Cars page and a **"Show me the in-budget cars →"** button on Start. All route through `setBudgetFilter('in')` / `toggleInBudget()` / `showInBudget()`; `updateFilterChrome()` keeps the chip in sync.

## Buyer context (do not invent beyond this)

- **Wants:** hybrid/EV, heated AND cooled seats ideally, simple non-overwhelming interface, SUV or sedan (sedan now fine — Harry drives an Accord, so a sedan-sized car feels familiar).
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
