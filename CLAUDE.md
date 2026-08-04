# Design Atlas — build & maintenance playbook

This repo is a **static, no-build-step reference site**. `index.html` is the hub; each
topic is one self-contained HTML file in `topics/`. GitHub Pages serves it directly.

Read this file before editing anything. It is the source of truth for structure,
design tokens, content conventions, and the QA that must pass before every commit.

---

## 1. What this is

A **Design Atlas**: a growing library of how design gets made, organised as

```
Design Atlas (hub)
  └─ Discipline  (e.g. Industrial Design)
       └─ Topic  (e.g. Surface & Substance)   ← a self-contained atlas page
            └─ Entry (a single concept: rule / how / tell)
```

The library grows **one topic at a time, by conversation.** The user proposes a topic;
you research it, draw its figures, build the page, wire it into the hub, run QA, commit.

## 2. Repo structure

```
index.html                       the hub — data-driven from DISCIPLINES/FUTURE arrays
topics/
  surface-and-substance.html     the reference implementation — copy this to start a topic
  <slug>.html                    one file per topic, fully self-contained
CLAUDE.md                        this file
README.md                        human-facing overview
```

There is **no framework, no bundler, no package.json required to ship.** Node is used
only for local QA (see §7). Everything runs by opening the HTML.

## 3. Design tokens — identical across every page

Every page defines these in `:root`. Do not drift from them; the family resemblance is
the point.

```css
--ink:#1a1b1e;      /* page background — the neutral-grey viewing booth  */
--booth:#212328;    /* card / panel background                          */
--booth-2:#282b31;  /* raised / hover panel                             */
--steel:#3a3f47;    /* hairlines, borders                               */
--steel-2:#4c525c;  /* secondary lines, dashed guides                   */
--vellum:#e9e7e2;   /* primary text                                     */
--vellum-dim:#9aa0a8;  /* secondary text                                */
--vellum-mute:#6f767f; /* labels, captions                             */
--comb:#ff4d8d;     /* accent 1 — "wrong / attention" (curvature comb)  */
--iso:#4fd6c0;      /* accent 2 — "right / signal" (isophote)           */
--draft:#f5b544;    /* accent 3 — dimensions, warnings                  */
```

Fonts (Google Fonts, loaded per page):
- **Archivo** 500–800 — display / headings
- **Karla** 400–600 — body
- **JetBrains Mono** 400–700 — labels, code, figure captions, stats

Voice for accent colours in figures: `--iso` = the correct/continuous case,
`--comb` = the defect/attention case, `--draft` = measured numbers and cautions.

## 4. The topic page — reference implementation

`topics/surface-and-substance.html` **is the template.** To build a new topic, copy it
and replace the content, keeping the engine. Its anatomy:

- **Masthead** with a back-link `<a class="atlas-back" href="../index.html">`, an eyebrow,
  and a title (`<h1>` with an `<em>` accent word).
- **Optional signature interactive** (S&S has the continuity lab). Bespoke per topic —
  a topic earns a hero interactive when one concept anchors the whole subject. Not required.
- **Controls**: a search box (`#q`) and domain filter chips, plus a "niche only" toggle.
- **`main`** rendered from data.
- Three data arrays + a figure library, then a render IIFE.

### 4a. Entry schema

Entries live in the `ENTRIES` array. Each is one concept:

```js
{
  t:`Draft angle`,          // title
  d:`dfm`,                  // domain id (must exist in DOMAINS)
  n:0,                      // niche flag: 0 = fundamental, 1 = niche/advanced
  fig:`draft`,              // OPTIONAL — key into FIG (an inline SVG diagram)
  img:[`File.jpg`,`what it shows`,`credit string`],  // OPTIONAL — a photo (see §6)
  src:[[`Label`,`https://url`], ...],                // OPTIONAL — "learn more" links
  one:`one-line definition.`,
  rule:`the governing rule or number, terse.`,
  data:[`verified figure 1`,`verified figure 2`],    // OPTIONAL — sourced specifics
  how:`how it is actually done in practice.`,
  tell:`the visual tell — how you spot it, or spot it done wrong.`,
  also:`cross-references · synonyms`                  // OPTIONAL
}
```

An entry should have **at most one** of `fig` / `img`. If it has neither, give it `src`
so it still carries a "Learn more". The render shows a badge per card: `FIG`, `PHOTO`,
or `REF`.

`DOMAINS` is `[id, 'Display Name', 'blurb']` rows — the sections a topic is divided into.

### 4b. Content principles

- **Structure every entry as rule → how → tell.** The rule is the number you open a
  conversation with; the how is practice; the tell is what the eye catches. This triad is
  the whole value of the atlas — keep it.
- **Numbers get sourced.** Anything quantitative goes in `data` with a real standard or
  reference behind it, and ideally a `src` link. Never invent a figure. If unsure, say so
  in the text and flag it, or leave it out.
- **Prose, not marketing.** Terse, concrete, a practitioner talking to a peer. No hype.
- **Evenhanded on contested points**; present the trade-off, not a verdict.
- **Niche flag** (`n:1`) for anything past the fundamentals, so the "niche only" filter works.

## 5. Figures (inline SVG) — the house style

Figures are entries in the `FIG` object: `FIG.name = \`<svg ...>...</svg>\``. They use CSS
classes defined once in the page's `<style>`:

- Line/label tiers: `s-lbl` / `s-lbl-a` / `s-lbl-g` / `s-lbl-d` at **9.5px**
  (neutral / comb / iso / draft), and a smaller caption tier
  `s-cap` / `s-cap-a` / `s-cap-g` / `s-cap-d` at **7.8px**.
- Shape helpers: `s-fill`, `s-line`, `s-thin`, `s-dim` (dashed dimension), etc.
- Colours come **only** from the CSS variables, never hard-coded hex, so figures re-theme
  with the page.

### 5a. The text-overflow budget (this bites — check it every time)

SVG text does not wrap. JetBrains Mono advance width is ~**0.602 × font-size**, so:
- 9.5px labels: **~5.72 px/char** — `x + len(text)*5.72` must stay `< viewBox width`.
- 7.8px captions: **~4.70 px/char**.

After writing/generating figures, run the overflow audit in §7. Fix by shortening the
line, splitting it into two `<text>` rows, or demoting a `s-lbl` to `s-cap`. Also confirm
every `y` is inside the `viewBox` height.

### 5b. Computed figures

When a diagram represents real geometry (splines, curvature, kinematics, data), **compute
it** rather than eyeballing it. S&S generated its NURBS and continuity figures from a real
Cox–de Boor evaluator in Python, emitted SVG paths, and verified properties numerically
(e.g. a w = 1/√2 weight reproduces a circle to 1e-16). Prefer this: it is correct by
construction and the verification doubles as a test. Keep any generator scripts in a
`tools/` dir if you add them; they are dev-only and need not ship.

## 6. Images & sources

- **Photos**: hotlink from Wikimedia Commons via the permanent-redirect endpoint
  `https://commons.wikimedia.org/wiki/Special:FilePath/<File_Name.ext>`. Always add an
  `onerror` fallback panel (see the `photoHTML` function in the reference page) so a broken
  image degrades to a written description instead of a hole. Credit every image; mark any
  licence you have not personally confirmed as "verify on the file page".
- **Prefer source links to photos for abstract concepts.** A `src` link to a durable
  reference (Wikipedia, a standards body, a regulator, a manufacturer's technical page)
  is more useful and never rots the way a hunted-down image does.
- **Only hotlink licence-safe sources.** Wikimedia PD / CC-BY / CC-BY-SA. Do not hotlink
  arbitrary web images.

## 7. QA — must pass before every commit

Run from the repo root. `jsdom` is the only dev dependency (`npm i -D jsdom` once).

1. **JS syntax** — extract every `<script>` body and `node --check` it.
2. **Tag balance** — exactly one each of `<html> </html> <body> </body> <style> </style>`,
   and the expected count of `<script>` blocks.
3. **jsdom smoke test** — load the page, confirm: all entries render as cards, section
   count matches `DOMAINS`, search filters, a detail panel opens with its figure/photo/src,
   and the masthead counters are right.
4. **Figure overflow audit** — for every `FIG`, no `<text>` exceeds its `viewBox` in x
   (using the per-char budgets in §5a) or y.
5. **Reference integrity** — every `fig:` key exists in `FIG`; no `FIG` is orphaned;
   every `src`/`img` URL is well-formed.

A convenience script lives at `tools/qa.mjs` if present; otherwise the checks above are
short to inline. **Do not commit a page that fails any of them.**

## 8. Adding a topic — the recipe

1. Agree the topic and its `DOMAINS` (its internal sections) with the user first.
2. `cp topics/surface-and-substance.html topics/<slug>.html`.
3. Strip the S&S-specific content: replace masthead title/eyebrow, `DOMAINS`, `ENTRIES`,
   and `FIG`. Keep the engine (controls, render IIFE, photo/src machinery, CSS).
   Remove the continuity-lab section unless the new topic has its own hero interactive.
4. Write entries (§4) and figures (§5); compute any geometric ones (§5b).
5. Wire into the hub: in `index.html`, add a topic object to the Industrial Design
   discipline's `topics` array (before the `{status:'add'}` card), with `slug`, `title`,
   `status:'live'`, `url`, `tagline`, and `stats`. To open a **new discipline**, add a
   discipline object to `DISCIPLINES` and, if it was in `FUTURE`, remove it there.
6. Run all of §7 on the new page **and** re-smoke the hub.
7. Commit with a clear message (§9). Pages redeploys automatically.

## 9. Commit & deploy

- Branch: **main**. GitHub Pages serves `index.html` at the site root and
  `topics/<slug>.html` at that path. **The root file must be named exactly `index.html`**
  (lowercase, single `.html` — a stray `index.html.html` is the classic 404 cause).
- One logical change per commit. Messages: `add topic: <name>`, `hub: <change>`,
  `s&s: <change>`, `fix: <what>`.
- After pushing, the site is live in ~1 min; check the Actions/Pages deployment is green.

## 10. Guardrails

- **Never break a working topic page** while adding another. Edit one file at a time; the
  self-contained structure exists precisely so a mistake can't cascade.
- **Do not over-genericise.** Resist merging all topics into one templated engine — each
  topic keeps its own bespoke figures and any hero interactive. Shared identity lives in
  the tokens (§3) and this playbook, not in shared code.
- **No browser storage** (`localStorage`/`sessionStorage`) — keep state in JS memory so
  pages work anywhere.
- **Keep it offline-friendly** apart from Google Fonts and hotlinked photos. Both degrade
  gracefully (system fonts; image fallback panels).
