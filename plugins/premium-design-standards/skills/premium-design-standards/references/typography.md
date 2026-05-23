# Typography (Hebrew + English)

The single fastest way to look generic is a default typeface. Type is a decision; make it.
Fonts are tagged **[free]** (installable/renderable now — use as defaults) or
**[commercial]** (aspirational upgrade — only if licensed and installed).

## Hebrew faces

**Defaults [free]:**
- **Frank Ruhl Libre** — the canonical Hebrew editorial serif (Fontef revival, Google Fonts).
  High-contrast, authoritative. **Headline/display only** — degrades below 16px.
- **Maaravon** (AlefAlefAlef) — traditional serif, round geometric counters; good editorial **body**
  when Frank Ruhl is too heavy.
- **Ploni / Almoni** (AlefAlefAlef, free tiers) — contemporary sans; Ploni for text hierarchy,
  Almoni more geometric/display.
- **Heebo / Assistant** — **fallbacks only.** Technically fine but generic ("the Inter of Hebrew");
  if forced to use, pair with a distinctive display face and never rely on them for identity.

**Upgrade picks [commercial]:**
- **GT America Intl Hebrew** (Grilli Type / Fontef) — best **single bilingual system**; Hebrew
  and Latin share weight distribution, so mixed lines look designed-as-one. First choice when budget allows.
- **Narkiss Block / Narkiss Text / Narkissim** (Fontef) — distinctly Israeli premium; Block is the
  iconic geometric display, Text/Narkissim for body.
- **ABC Favorit Hebrew** (Dinamo) — contemporary tech/editorial; Display vs Standard optical cuts.
- **Postea Hebrew** (TypeTogether, 2024) — corporate/branding/signage.

**BAN:** David / Miriam / Guttman (Windows system fonts — zero intent, broken digital rendering).
Heebo/Rubik/Assistant as a *primary identity* face.

## Latin faces

**Defaults [free]:**
- Serif display: **Fraunces** (variable, optical sizing, soft/wonky axes) or **Newsreader** (editorial, on Google Fonts).
- Sans: **Geist** (Vercel), **Hanken Grotesk**, **General Sans** (Fontshare) — grotesques with more character than Inter.

**Upgrade picks [commercial]:**
- Serif display: **Canela** (flared, editorial), **PP Editorial New** (high-contrast contemporary), **Freight**, **Lyon** (has a Hebrew companion).
- Sans: **Neue Montreal**, **Neue Haas Grotesk / Helvetica Now**, **GT America**, **Forma DJR** (true optical sizes: Display/Text/Micro).

**BAN:** Inter-as-the-only-font, Roboto, Open Sans, Lato, Source Sans, Montserrat, Poppins.
(Inter is fine as a UI utility — just not as your display face or your only decision. If you must,
use **Inter Display** for headings, not one weight throughout.)

## Type scale & hierarchy

- Modular ratio: **1.25** (Major Third) for dense UI; **1.333** (Perfect Fourth) for editorial / video titles.
- Weight contrast must be extreme: pick **two** weights — body 400, emphasis 700 or 900 — and let **size**
  do the rest. **Never 4 weights in one layout.** Never lean on 400-vs-500/600 "contrast."
- Use optical sizing where the face supports it; set `font-optical-sizing: auto`. At ≤12px use a
  Text/Micro cut, not a scaled-down Display cut.

## Hebrew typesetting rules

- **Line-height:** 1.5–1.6 for body; 1.1–1.2 for display (the lamed ascender clips below ~1.0).
  Mixed He/En paragraphs: 1.6 floor (Hebrew sits in a tighter vertical band; Latin baselines mismatch).
- **Never apply positive letter-spacing to Hebrew.** It breaks the connected word-image and misaligns
  niqqud. At 72px+ display you may *tighten* −0.01 to −0.02em, never widen.
- **Punctuation:** use geresh `׳`, gershayim `״`, maqaf `־` — not Latin `'`, `"`, `-`. Check glyph coverage.
- **Niqqud:** only with fonts that have anchored niqqud (Frank Ruhl Libre, Narkiss Text, most Fontef);
  avoid in display unless OpenType support is confirmed.
- **Numerals:** modern Hebrew uses Western digits 0–9, written LTR inside the RTL line — the BIDI
  algorithm handles this when direction is set correctly.

## Mixed Hebrew/English (bidi)

- Always set base direction on the container: `html[lang="he"] { direction: rtl; }`.
- Wrap inline LTR runs (brand names, numbers, code) in `<bdi>` or `unicode-bidi: isolate` so they
  don't reverse. **Never** `unicode-bidi: bidi-override`.
- **Font strategy:** prefer a single bilingual face (GT America Intl, ABC Favorit Hebrew) — no
  matching needed. If pairing two fonts, optically match the Hebrew mem-height to sit **between** the
  Latin x-height and cap-height (adjust ±1–2px by eye, not arithmetic).
- **Headlines** are the hardest case: set outer `dir` to the dominant language, wrap the minority run
  in `<bdi>`/`<span dir="ltr">`, avoid punctuation at the direction boundary, and **test in a bidi
  visualizer** — if any character is out of order vs. what you typed, fix the markup.
- See `[[rtl-design]]` for full RTL implementation and `[[hebrew-content-qa]]` for copy QA.

```css
html[lang="he"] { direction: rtl; }
```
```html
<p dir="rtl">המוצר נקרא <bdi>iPhone 15</bdi> וזה חשוב</p>
```

## Typography in video (1920×1080)

- **Min sizes:** body/subtitle 40–60px · lower-thirds 48px+ · titles 72–120px. Below 40px competes with compression artifacts. (4K native: double; 4K→1080p delivery: keep.)
- **Dwell:** ≥13 characters/second → a 30-char line needs ≥2.3s **stationary**. Animation-in time doesn't count as reading time.
- **Limits:** ≤30 chars/line, ≤3 lines on screen. (More in `readability.md`.)
- **Motion:** animate **IN → HOLD static → cut**. Word stagger 60–80ms (not per-letter — per-letter reads as decoration). Expo ease-out `cubic-bezier(0.16,1,0.3,1)`; **no bounce on body text**. Animate `transform: scale()`, never `font-size`.
- **Over footage:** always a scrim or shadow `0 2px 8px rgba(0,0,0,0.6)` minimum — never float light text on variable backgrounds.
- **Hebrew in motion:** RTL text enters from the **right**; stagger word reveals **right-to-left** (first word is rightmost). Confirm character order in the actual rendered frame, not just the dev preview.

## Sources
Fontef / Grilli Type / Dinamo / TypeTogether / AlefAlefAlef foundry pages · Meir Sadan
"An Introduction to Hebrew Type" · W3C inline-bidi-markup · TypeTogether multiscript guide ·
legibility.info (text in video) · type-scale.com · Typewolf · Pangram Pangram pairings · DJR Forma.
