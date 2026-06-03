# Car Guide — Claude Code Session Brief

## Project Overview

A single-file HTML car shopping guide built for Harry's girlfriend who is moving to San Diego and buying/leasing her first car (~September 2026). The file is hosted on GitHub Pages at a shareable URL so she can open it on her phone.

**File:** `index.html` (renamed from `car-guide.html` for GitHub Pages)

---

## Architecture — Critical to Understand

### Zero-JavaScript Design
The entire UI is driven by **pure CSS using the checkbox/radio hack**. There is **no JavaScript** anywhere in the file. This was intentional — the file was originally being opened via `file://` protocol on mobile which blocks inline scripts in some browsers. Now that it lives on GitHub Pages (HTTPS), JS would work fine, but the CSS-only approach is clean and should be preserved or carefully extended.

### How Navigation Works
Five tab pages are controlled by **radio inputs** at the top of `<body>`:
```html
<input type="radio" name="tab" id="tab-cars" checked>
<input type="radio" name="tab" id="tab-compare">
<!-- etc. -->
```
CSS then shows/hides `.page` divs using the general sibling combinator `~`:
```css
#tab-cars:checked ~ #page-cars { display: block }
```
Tab pill labels are `<label for="tab-cars">` elements styled to look like buttons.

**Important:** All hidden `<input>` elements **must stay at the very top of `<body>`**, before any content they need to control via CSS siblings. Moving them breaks everything.

### How the Vent Toggle Works
A single checkbox `#vent-chk` (checked by default = "show vent options") controls two content layers:
```css
#vent-chk:checked     ~ * .vent-only { display: block }
#vent-chk:not(:checked) ~ * .vent-only { display: none }
#vent-chk:not(:checked) ~ * .no-vent  { display: block }
```
The `* ` (universal child combinator) is needed because the content is nested inside `.page` divs which are siblings of the checkbox, not direct children.

The toggle visual is a `<label for="vent-chk">` styled as an iOS-style switch. State (ON/OFF text, switch position, notice banner swap) all driven by CSS.

### How Card Expand/Collapse Works
Each car card has its own dedicated checkbox at the top of `<body>`:
```html
<input type="checkbox" id="exp-rav4-lim" class="card-expand-chk">
```
Inside the card, a `<label for="exp-rav4-lim">` acts as the expand button. The `.card-details` div follows the label and uses:
```css
.card-expand-chk:checked ~ .card-details { display: grid }
```
**Problem:** Because the inputs are at the top of `<body>` and the cards are deeply nested, this sibling selector doesn't work directly. **This is a known bug in the current file** — the card expand/collapse may not function correctly. Fixing this requires either:
- (A) Moving the checkbox inputs inside each card (breaks vent toggle global approach)
- (B) Using the `:has()` pseudo-class (modern CSS, good browser support as of 2024+)
- (C) Adding minimal JavaScript just for the card toggles (acceptable since we're on GitHub Pages now)

---

## Content Structure

### Pages (5 tabs)
1. **Cars** (`#page-cars`) — Car cards with pricing. Has two layers:
   - `.vent-only` — RAV4 Hybrid Limited, Tucson Hybrid Limited, Tesla Model 3 LR
   - `.no-vent` — RAV4 XLE Premium, Tucson SEL Convenience, RAV4 XLE/LE
2. **Compare** (`#page-compare`) — Side-by-side table + 5-year cost snapshot. Also has `.vent-only` / `.no-vent` variants.
3. **Insurance** (`#page-ins`) — Rate estimates + insurer tips
4. **Leasing 101** (`#page-lease`) — Full lease explainer (who pays what, wear/tear, lease vs buy)
5. **Tips** (`#page-tips`) — 8 shopping tips for September 2026

### Car Data (verified against real sources, June 2026)

#### WITH VENTILATED SEATS (vent-only layer)
| Car | MSRP (top trim) | Buy New CA all-in | CPO/Used | Lease est/mo | Notes |
|-----|-----------------|-------------------|----------|--------------|-------|
| 2026 RAV4 Hybrid Limited | $44,750 | ~$48–51K | $30–38K | ~$500–540 | Cooled seats Limited only. AWD standard. 43/37 MPG. |
| 2026 Tucson Hybrid Limited | ~$42,075–45,025 | ~$47–50K | $33–39K | ~$442–500 | Physical climate dials. Best interface. 38/38 MPG. |
| Tesla Model 3 LR RWD | ~$42,490 | ~$44–47K | $28–36K | ~$400–470 | Sedan only. No buy at lease end. Best lease price. |

#### WITHOUT VENTILATED SEATS (no-vent layer)
| Car | MSRP | Buy New CA all-in | CPO/Used | Lease est/mo | Notes |
|-----|------|-------------------|----------|--------------|-------|
| 2026 RAV4 Hybrid XLE Premium | ~$37,550 | ~$39–43K | $26–33K | ~$400–440 | Heated only. Moonroof, SofTex leather. |
| 2026 Tucson Hybrid SEL Convenience | ~$36–38K | ~$38–42K | $27–33K | ~$390–430 | Heated only. Bose audio. Physical climate. |
| 2026 RAV4 Hybrid XLE/LE | ~$34,750–36K | ~$36–39K | $23–30K | ~$350–400 | Most affordable. LE lease $399/mo $0 down nationally. |

#### Insurance estimates (San Diego, new solo policyholder, full coverage required)
- RAV4/Tucson hybrid: ~$200–267/mo ($2,400–3,200/yr)
- Tesla Model 3: ~$267–367/mo ($3,200–4,400/yr) — significantly higher due to repair costs
- SD city average all cars: ~$233/mo ($2,800/yr)
- After 1 year with record: ~$183–233/mo
- Cheapest insurers in SD: Geico (~$1,203/yr ref), Mercury (~$1,300), Progressive (~$1,509), USAA (~$1,312, military only)

---

## Design System

### Colors (CSS variables)
```css
--cream: #faf7f2        /* page background */
--charcoal: #1a1a1a     /* header, dark sections */
--mid: #444             /* body text */
--soft: #777            /* secondary/muted text */
--accent: #c17d3c       /* warm orange — eyebrows, highlights */
--accent-light: #e8c99a /* pale gold — hero text accent */
--accent-pale: #fdf4e7  /* very pale orange — highlighted price cells, notices */
--green: #3d7a5c        /* positive, included features, best picks */
--green-light: #d4ede2  /* green badge backgrounds */
--blue: #2d5fa6         /* links, expand buttons, Tesla */
--blue-light: #dce8f7   /* blue badge backgrounds */
--red: #c04040          /* warnings, unavailable features */
--red-light: #fce8e8    /* red badge backgrounds */
--border: #e0d8cc       /* all card/table borders */
```

### Typography
- **Display/headings:** `Playfair Display` (serif, from Google Fonts) — weights 700, 900
- **Body:** `DM Sans` (sans-serif, from Google Fonts) — weights 300, 400, 500, 600
- Base font size: 15px
- Section eyebrows: 10px, 600 weight, 0.16em letter-spacing, uppercase, `--accent` color
- Section h2: Playfair Display, 22px, 700
- Section sub: 13px, `--soft` color

### Component Patterns

**Car card structure:**
```
.car-card
  .card-head (flex row: info left, mpg right)
    .card-badge (pill)
    .card-name (Playfair)
    .card-sub
  .card-mpg
  .price-row (3-column grid)
    .price-cell (x3, last one .hi for accent highlight)
  .card-toggle-lbl (expand button — label element)
  .card-details (2-col grid, hidden by default)
    features column
    notes column
```

**Dark section (leasing page):**
```
.dark-section (charcoal bg, rounded)
  .dark-head (colored header strip)
  .lease-item (flex row: icon + text) × N
```

---

## Known Issues / TODO

### 1. Card expand/collapse is broken (HIGH PRIORITY)
The card detail checkboxes are at the top of `<body>` but the cards are deeply nested. CSS `~` sibling selector can't reach through nested containers. **Recommended fix:** Add a small `<script>` block at the bottom of body — now that we're on GitHub Pages, JS works fine:

```javascript
document.querySelectorAll('.card-toggle-lbl').forEach(function(lbl) {
  lbl.addEventListener('click', function() {
    var details = this.nextElementSibling;
    var isOpen = details.style.display === 'grid';
    details.style.display = isOpen ? 'none' : 'grid';
    this.querySelector('.chevron').textContent = isOpen ? '▼' : '▲';
  });
});
```
And remove the `id` attributes from the label `for` attributes for card toggles (or keep them — either works).

### 2. Pill nav active state on mobile — scroll into view
When a tab is selected, the active pill should scroll into view in the horizontal scroll container. This requires JS — the CSS-only approach can't scroll a container to a specific child. A small scroll handler would improve UX on mobile:
```javascript
document.querySelectorAll('.pill-nav label').forEach(function(lbl) {
  lbl.addEventListener('click', function() {
    this.scrollIntoView({ behavior: 'smooth', block: 'nearest', inline: 'center' });
    window.scrollTo({ top: 0, behavior: 'smooth' });
  });
});
```

### 3. Sticky nav offset
The toggle banner is `position: sticky; top: 0` and the pill nav is `position: sticky; top: 53px`. If the toggle banner height changes (e.g. on very small screens where the label wraps), the pill nav offset will be wrong. Consider using JS to dynamically set `top` on the pill nav, or use a CSS custom property updated via JS.

### 4. Card details grid on mobile
At `max-width: 400px`, `.card-details` switches to `grid-template-columns: 1fr`. The breakpoint may be too narrow — consider bumping to `max-width: 600px` for phones in landscape or larger-screen phones.

### 5. Vent toggle and compare table
The compare page has both `.vent-only` and `.no-vent` table variants. They work correctly with the toggle. If adding more content to the compare page, maintain this pattern.

---

## File Deployment

**GitHub Pages setup:**
1. Repo name: `car-guide` (or whatever was chosen)
2. File must be named `index.html` in the repo root
3. Settings → Pages → Deploy from branch → main → / (root)
4. URL: `https://[username].github.io/car-guide`

**To update:** Edit `index.html`, commit, push. GitHub Pages auto-deploys in ~30–60 seconds.

---

## Context: What This Is For

- **Who:** Harry's girlfriend, moving to San Diego from out of state (~September 2026)
- **Budget:** ~$30K to buy, or leasing (monthly payment separate from buy budget)
- **Wants:** Hybrid, SUV preferred (CRV/RAV4/Pilot size range — not tiny like EcoSport), heated AND cooled/ventilated seats ideally, simple non-overwhelming interface
- **Doesn't want:** Kia (personal preference), anything too small
- **Coming off parents' insurance** — will need first solo policy, expect ~$200–280/mo to start
- **Purchase location:** Must buy/register in California. San Diego area or SoCal (Temecula, El Cajon, etc. all fine)
- **Timing:** ~3 months from now (September 2026). End-of-quarter timing is advantageous for negotiating.

---

## Session Goals (suggested starting points)

1. **Fix card expand/collapse** — add the small JS block described above
2. **Add pill scroll-into-view** — improve mobile UX when switching tabs
3. **Test and fix any CSS sibling selector issues** caused by nesting
4. **Optional:** Add a simple "monthly cost calculator" input on the Compare page (lease mo + insurance = total monthly) — could be a simple JS form
5. **Optional:** Add anchor links or a "back to top" button for long pages (Leasing 101 is the longest)
