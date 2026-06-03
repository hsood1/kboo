# NYC Pizza Crawl — Project Brief for Claude Code

This document is the full context for turning the existing scrapbook page into a
published **GitHub Pages** site. Read it end to end before changing anything.

There is already a finished, working file: **`index.html`**. It is fully
self-contained (all images are embedded as base64; there is **no JavaScript** and
**no external file dependency** except web fonts). It can be deployed to GitHub
Pages as-is. Everything below documents what it contains and how to extend it.

---

## 1. Goal

Publish a one-page, print-friendly digital "scrapbook" of a 3-day NYC pizza (and
bagel) crawl as a GitHub Pages site. The vibe is a warm, hand-made scrapbook:
cream paper, taped instant-photo prints, handwritten captions, two playful
toggles. It is meant to be viewed in a browser **and** printed for a physical
scrapbook.

Primary success criteria:
1. Loads correctly as `https://<user>.github.io/<repo>/` (or a custom domain).
2. Renders correctly **even in viewers that do not run JavaScript** (this was a
   hard requirement — see §7). Do not reintroduce JS-dependent rendering.
3. Prints cleanly (one continuous page; cards never split across page breaks).

---

## 2. Current state / what exists

- `index.html` — the entire site in one file:
  - Inline `<style>` (no external CSS).
  - 15 photos embedded as base64 JPEG data URIs (600×600, letterboxed on black).
  - 2 inline SVG "doodles".
  - **No `<script>` tag.** Both interactive controls are pure CSS.
- The page works offline except for fonts, which load from Google Fonts. Without
  internet the layout is intact but falls back to default serif/sans fonts.

---

## 3. Local preview

```bash
# any static server works; e.g.
python3 -m http.server 8000
# then open http://localhost:8000/
```

---

## 4. Deploy to GitHub Pages

Project-site approach (simplest):

```bash
git init
git add index.html NYC.md
git commit -m "Initial NYC pizza crawl scrapbook"
git branch -M main
git remote add origin git@github.com:<user>/<repo>.git
git push -u origin main
```

Then in the GitHub repo: **Settings → Pages → Build and deployment →
Source: "Deploy from a branch" → Branch: `main` / root (`/`) → Save.**
The site publishes at `https://<user>.github.io/<repo>/` within a minute or two.

Notes:
- GitHub Pages serves `index.html` at the root automatically.
- Add an empty `.nojekyll` file at the repo root to be safe (prevents Jekyll from
  touching any future `_`-prefixed asset folders).
- Custom domain (optional): add a `CNAME` file containing the domain and set the
  DNS records per GitHub's docs, then set the custom domain under Settings → Pages.

---

## 5. Recommended repo structure

The current single file is fine to ship. If you refactor for maintainability,
this is the suggested layout (keep `index.html` working at every step):

```
/
├── index.html          # entry point
├── NYC.md              # this brief
├── README.md           # short public-facing readme (create)
├── .nojekyll
├── assets/
│   ├── styles.css      # extracted from the inline <style> (optional)
│   └── images/         # extracted base64 -> real .jpg files (optional)
└── CNAME               # only if using a custom domain
```

### Extracting the embedded images (optional)

Each photo is a `data:image/jpeg;base64,...` URI inside an `<img>`. To pull them
into real files:

```python
import re, base64, pathlib
html = pathlib.Path("index.html").read_text()
pathlib.Path("assets/images").mkdir(parents=True, exist_ok=True)
for i, m in enumerate(re.finditer(r'data:image/jpeg;base64,([A-Za-z0-9+/=]+)', html)):
    pathlib.Path(f"assets/images/img_{i:02}.jpg").write_bytes(base64.b64decode(m.group(1)))
```

Then replace the data URIs with `assets/images/...` paths. The asset key → meaning
map is in §9 so you can give them sensible filenames.

---

## 6. Content spec (source of truth)

### Trip facts
- **Dates:** May 31 – June 2, 2026 (3 days, two stops per day).
- **Two eaters / two rankings:** referred to throughout as **His** and **Hers**.
  (His = the site owner.) The page never uses real names — keep it "His/Hers".
- Tagline / kicker: `A New York Slice Story`.
- Title: `The Pizza Crawl` ("Pizza" in tomato red, slightly rotated).
- Blurb: *"Six legendary slices, three summer days, two stops a day — two very
  different rankings, and a serious bagel habit on the side."*
- Scoreboard stats: **6** Pizzerias · **3** Bagel Shops · **3** Days · **2** Full Bellies.
- Footer: *"Eaten & ranked — his & hers — all over Manhattan ❤ Summer 2026"*.

### The pizzerias (this is the **visit order** = STOP order on the page)

| Stop | Name | Neighborhood label | Date | His # | Her # |
|---|---|---|---|---|---|
| 1 | Village Square Pizza | Upper East Side | May 31 | 4 | 4 |
| 2 | Mama's Too | Upper West Side | May 31 | 3 | 1 |
| 3 | Koronet Pizza | A 10-min walk from the Met | June 1 | 2 | 3 |
| 4 | L'Industrie | West Village | June 1 | 1 | 2 |
| 5 | Joe's Pizza | Financial District | June 2 | 6 | 5 |
| 6 | Scarr's Pizza | Lower East Side | June 2 | 5 | 6 |

Per-card photo captions + handwritten note text:

- **Village Square Pizza** — photos: `vsq_sign` ("gourmet NYC pizza"),
  `vsq_person` ("where it all began").
  Note: *"Right on 82nd & Lex, where the whole crawl kicked off. The half-pesto,
  half-marinara grandma square — crispy fried bottom, big chili-flake energy."*
- **Mama's Too** — photos: `mama_sign` ("the sign"), `mama_tray` ("the whole tray").
  Note: *"Square game absolutely on point — a mushroom white, a hot-chicken number,
  and a flawless round margherita. The night-cap of day one."*
- **Koronet Pizza** — photos: `kor_decal` ("the jumbo slice"), `kor_slice` ("one slice").
  Note: *"“The jumbo slice since 1981.” I got the regular size... which is still the
  size of my face. Foldable, cheesy, gloriously oversized."*
- **L'Industrie** — photos: `lind_sign` ("the window"), `lind_slice` ("classic + burrata").
  Note: *"Two slices, both outstanding — a clean classic cheese and the famous burrata,
  with cold dollops of stracciatella over fresh basil. The burrata sealed the deal."*
- **Joe's Pizza** — photos: `joe_box` ("the box"), `joe_slice` ("plain · mozz · white").
  Note: *"The Greenwich Village institution since 1975. “You saw us in Spider-Man.” A
  reliable classic — plain, fresh mozz, and white slices."*
- **Scarr's Pizza** — photos: `scarr_sign` ("the storefront"), `scarr_slice` ("classic cheese").
  Note: *"House-milled flour, deeply savory sauce, and a clean classic cheese slice —
  with a lovely slight char around the edges of the crust."*

### The two rankings (used by the "The Ranking" table)

- **His:** 1) L'Industrie 2) Koronet 3) Mama's Too 4) Village Square 5) Scarr's 6) Joe's
- **Hers:** 1) Mama's Too 2) L'Industrie 3) Koronet 4) Village Square 5) Joe's 6) Scarr's

The #1 row in each table is highlighted gold with a ★.

### The bagels (day order; shown in "The Bagels" section)

| Day | Shop | Date | Badge | Photos / doodles |
|---|---|---|---|---|
| 1 | Liberty Bagels | May 31 | ★ Her #1 | photo `bagel_rainbow` ("rainbow bagel") + doodle ("the mess-up") |
| 2 | Ess-a-Bagel | June 1 | — | photo `bagel_ess` ("lox, egg & avo") + doodle ("eggs over avo") |
| 3 | Apollo Bagels | June 2 | ★ His #1 | photo `bagel_apollo` ("everything bagel") |

Bagel card notes:
- **Liberty Bagels:** *"Day one went bold — a rainbow bagel with strawberry cream
  cheese, plus our accidental “mess-up” order: a pumpernickel everything with olive
  and jalapeño cream cheese."*
- **Ess-a-Bagel:** *"The big guns: a loaded pumpernickel with lox, egg, and avocado,
  alongside my everything bagel with eggs over avo and olive cream cheese."*
- **Apollo Bagels:** *"A naturally-leavened everything bagel done two ways — half
  scallion cream cheese, half cultured butter & jam. Blistered, caramelized crust and all."*

### The doodles (inline SVG, `viewBox="0 0 240 240"`)
Two hand-drawn bagel illustrations stand in for un-photographed bagels, each tagged
with a small "✎ doodle" ribbon:
1. **"the mess-up"** — top-down pumpernickel everything bagel with two cream-cheese
   smears (olive = olive-green w/ dark bits; jalapeño = pale green w/ flecks).
2. **"eggs over avo"** — top-down everything bagel with mashed avocado, a fried egg
   (white + yolk), and an olive cream-cheese smear peeking out.

---

## 7. Technical implementation (and what NOT to break)

- **No JavaScript — by design.** The file was rebuilt to remove all JS because it
  is shared as a raw `.html` file and opened in apps that don't run scripts (mail
  previews, messaging apps). Keep all interactivity CSS-only.
  - **His/Hers toggle** = the "checkbox/radio hack":
    two hidden radios `#who-his` (checked by default) and `#who-her`, styled
    `<label>` pills, and **two pre-rendered tables** (`.his-table`, `.her-table`).
    CSS shows/hides via `#who-her:checked ~ .her-table { display:table }` etc.
  - **Bagel switch** = hidden checkbox `#bagelToggle` + a `<label class="switch">`.
    `.bagels { display:none }` by default; `#bagelToggle:checked ~ .bagels { display:block }`.
    **Bagels are intentionally hidden until the switch is flipped.**
- **Images:** square (600×600), letterboxed on a **black** background (no cropping —
  important so wide storefront signs aren't cut off). Displayed at 240×240 inside
  white "polaroid" frames with a washi-tape pseudo-element and a small rotation
  (`--r`). On mobile they shrink to 128×128.
- **Print:** `@page { margin:12mm }`, `print-color-adjust:exact`, cards use
  `break-inside:avoid`, entry animations are disabled in print.

---

## 8. Design system

CSS custom properties (in `:root`):

```
--paper:  #f6efe0   /* warm cream background */
--ink:    #241a12   /* near-black warm brown text */
--soft:   #5e4f3e   /* muted brown (captions, meta) */
--tomato: #c43a22   /* primary red accent */
--basil:  #46682f   /* green accent (Hers, bagels) */
--gold:   #bf9223   /* #1 / favorite highlights */
--crust:  #d99a4e   /* title shadow, dots */
--frame:  #fffdf6   /* polaroid frame white */
```

Fonts (Google Fonts):
- **Shrikhand** — display: the title, section headings, big stop/rank numbers.
- **Caveat** — handwriting: dates, sublines, photo captions, notes, doodle tags.
- **Fraunces** — serif body: pizzeria/shop names, blurb.
- **Libre Franklin** — uppercase labels: kicker, meta line, chips, scoreboard, pills.

Recurring components: `.spot` card (stop column + body), `.powhen` polaroid figure
(+ `.doodle` variant with `.doodlebox`), `.ranktable`, `.pill`, `.switch`,
`.chip.his` / `.chip.her`, `.favbadge`. Background has a faint SVG noise texture and
two soft radial color washes.

---

## 9. Image asset key → meaning

Used in the page (embedded as base64, in this order of appearance):
`vsq_sign` Village Square storefront sign · `vsq_person` person holding the grandma
square outside the shop · `mama_sign` Mama's Too lit sign at night · `mama_tray`
Mama's Too metal tray of three slices · `kor_decal` Koronet "Jumbo Slice Since 1981"
window logo · `kor_slice` Koronet cheese slice · `lind_sign` L'Industrie script
window logo · `lind_slice` L'Industrie classic + burrata slices · `joe_box` Joe's
white box logo · `joe_slice` Joe's three slices · `scarr_sign` Scarr's storefront ·
`scarr_slice` Scarr's cheese slice · `bagel_rainbow` Liberty rainbow bagel ·
`bagel_ess` Ess-a-Bagel lox/egg/avocado · `bagel_apollo` Apollo everything bagel.

---

## 10. Intentional decisions — DO NOT "fix" these

- **Koronet's neighborhood is intentionally labeled "A 10-min walk from the Met"**
  at the owner's request. (The famous Koronet is actually in Morningside Heights near
  Columbia. This is a deliberate personal label — leave it.)
- **Joe's is labeled "Financial District"** (the specific location visited). The box
  tagline reads "Greenwich Village Institution," but that prints on all Joe's boxes.
- **L'Industrie is "West Village"** (the Manhattan location, near the Friends
  building at Bedford & Grove) — not Williamsburg.
- **His & Hers rankings are exact** as listed in §6; Village Square and Koronet are
  swapped in Hers relative to an earlier draft — keep current values.
- **Bagels hidden by default**, revealed by the switch.
- **Captions are intentionally short**; the detailed descriptions live in the notes.
- **Keep it JavaScript-free.**

---

## 11. Suggested enhancement tasks (optional, for the Claude Code session)

Pick from these; none are required for a working deploy:
1. Add `<meta>` description + Open Graph / Twitter card tags and an `og:image`
   (export one of the slice photos) so link previews look good.
2. Add a `favicon` (a tiny pizza-slice or bagel SVG).
3. Create a short public `README.md` describing the project.
4. Optionally extract CSS to `assets/styles.css` and images to `assets/images/`
   (see §5) to shrink the HTML and make edits easier — keep the page identical.
5. Optionally **self-host the fonts** (download the four families, add `@font-face`)
   so it renders perfectly offline / in any viewer. This increases repo size.
6. Add a lightweight CSS-only lightbox (the checkbox hack again) to view photos
   larger — only if it can stay JS-free and print-safe.
7. Accessibility pass: ensure the radio/checkbox controls are keyboard-focusable and
   have visible focus states and proper `aria-label`s (the switch already has one).
8. Add a `print.css` or refine the print rules so a clean PDF export is one tidy
   document.

Whatever you change, re-verify the three success criteria in §1 (renders without JS,
deploys at the Pages URL, prints cleanly).
