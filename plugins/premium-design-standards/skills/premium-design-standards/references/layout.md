# Layout & Composition

Premium layout is **intentional structure**. Generic output has identifiable failures —
every rule here targets one. Numbers are hard defaults, not suggestions.

## Spacing — the 8pt system

Use multiples of 8 **exclusively**: `8, 16, 24, 32, 48, 64, 96, 128`.
Why 8: every common screen density (1×, 1.5×, 2×, 3×) maps to whole pixels, and it kills
micro-decisions. **4pt** exists for one thing only — sub-component micro-spacing (icon
padding, tight label-to-input). Never use 4pt to separate layout regions.

- 8 — tight related items (label↔input, icon↔text)
- 16 — component internal padding
- 24 — stacked elements within a section
- 32 — between components in a region
- 48 — major sections (mobile)
- 64 — major sections (desktop)
- 96 / 128 — hero breathing room

**BAN: 10, 12, 15, 20, 25, 30.** They break grid math and read as carelessness.

## Grid

12-column desktop → 8 tablet → 4 mobile (industry standard collapse).
- **1440px container:** 12 col, ~74px col width, 32px gutter, 80px side margins (1120px content).
- **1280px container:** 12 col, ~68px col, 24px gutter, 64px margins.
- **Tablet 768–1024px:** 8 col, 24px gutter, 32px margins.
- **Mobile <768px:** 4 col, 16px gutter, 16–20px margins.

**Baseline grid:** every line-height is a multiple of 8 (16px text → 24px lh; 48px display → 56px lh).
Generous outer margins read as premium; tight margins read as cheap (Müller-Brockmann).

## Composition

- **Rule of thirds** — primary focal at the power points, not center. At 1920×1080: vertical
  lines 640/1280, horizontal 360/720; intersections (640,360)(1280,360)(640,720)(1280,720).
  Center (960,540) is the lazy default.
- **Golden ratio (1.618)** for proportioning, not spirals: content split 61.8/38.2 ≈ 8+4 or
  7+5 columns; heading size ≈ body × 1.618; outer margin ≈ inner padding × 1.618.
- **Asymmetry is the premium default.** Symmetry only for semantic duality/authority
  (comparison, formal/institutional). Balance a heavy element against **open space**, not a
  mirrored twin — white space has visual weight.
- **Negative space is active.** Content ≤60% of the frame on most scenes; ≥40% empty on
  hero/title scenes; ≥16px (prefer 24px) clearance between any two elements; nothing touches edges.

## Visual hierarchy — five levers, in impact order

1. **Size** — most powerful. The dominant element must be ≥1.25× the next; if the two largest
   sizes are within 1.25× of each other, hierarchy collapses.
2. **Weight** — dramatic only: 300 vs 800/900. 400-vs-600 is imperceptible at reading speed.
3. **Color/contrast** — ≤3 tiers (100% / 70–80% / 40–50% opacity or muted value).
4. **Position** — top-left reads first (Latin); proximity groups.
5. **Spacing** — more space around an element elevates it.

**Squint test:** blur the comp (or Gaussian 10px). What you can still read is your hierarchy.
If two importance levels are indistinguishable when blurred, the hierarchy failed.

### Type scale reference

- **Video (1920×1080):** Hero 72–120px · Title 48–64px · Section 36–48px · Body/subtitle 24–32px
  · Caption 18–20px. Never below 16px on screen.
- **UI (desktop):** Display 40–64px · H1 32–40px · H2 24px · H3 20px · Body 16px · Small 14px.

## Video safe areas (1920×1080)

- **Action-safe** 93% — ~67px L/R, ~38px T/B. All important visuals inside.
- **Title-safe** 90% — ~96px L/R, ~54px T/B. **Every word of text inside this.**
- Social-cropped delivery (9:16/1:1 from 16:9 master): keep critical text in the center 60%.

## Patterns — BAN vs PREMIUM

**BAN (generic / AI-template tells):**
- The "Bootstrap holy trinity": centered hero headline → centered subhead → 3 equal cards.
- Symmetric everything; equal-width/height card grids (3-up/4-up identical).
- Uniform vertical-rectangle section stacking (hero→features→pricing→CTA, all same height).
- Glassmorphism + purple→blue gradient + floating blobs.
- Inter/Roboto/Poppins for display type.

**PREMIUM:**
- **Bento done right** — asymmetric cells, size = content importance (one dominant 2×2, supporting 1×1).
- **Editorial asymmetric splits** — 4+8 or 5+7 columns, image full-bleeds to one edge, text in the narrow column.
- **Full-bleed + type overlay** — image fills frame, text sits in it with a scrim or in a clear area.
- **Intentional grid-break** — exactly ONE element violates the otherwise-consistent grid (bleed, overlap, 3× size).
- **Single-column luxe** — 600–700px max-width, 80–120px section padding, large type. Looks expensive because it is restrained.
- **Overlap / layering** — type over imagery, elements exiting their container → depth without 3D.

## Video composition specifics

- Subject on a thirds line with **look space** open toward the empty 2/3.
- Text and focal subject never compete on the same third. Full-bleed centered subject → text gets a dedicated zone + scrim.
- **Lower thirds:** ≤20% frame height, 1–2 lines, 2 colors max, sans-serif (serifs flicker at video bitrates),
  slide-in-from-left OR fade-up only (never spin/bounce), 0.3–0.5s ease-out in, ≥3s hold, 0.2s out.
- **Depth via 4 parallax layers:** static bg → 0.1–0.2× → subject 1× → foreground 1.5–2×.
  Atmospheric perspective: distant elements lower-contrast, more blurred, less detailed.
- Establish hierarchy in the **first frame** of a scene; all text fully on screen by the **50% mark** of scene duration.
- **Mid-scene drift** — after entrance, elements keep doing something subtle. A static hold = slideshow.
- **95% hard cuts.** Dissolves/shader transitions only for major emotional beats. Every-cut-the-same = template.

## Density & alignment

- High density isn't bad (NYT print is dense) **if** gutters are consistent, baselines align, hierarchy is unambiguous.
- The real failure is **inconsistent** density — some areas crammed, others sparse, with no reason.
- **Alignment discipline:** every element aligns to something. When you break alignment, it must be the
  only thing breaking it in the scene, and it must be the most important element.

## Sources
spec.fm 8-point grid · designsystems.com space/grids/layouts · NN/g visual hierarchy ·
Müller-Brockmann *Grid Systems in Graphic Design* · Wikipedia Safe Area (television) ·
Studiobinder lower-thirds · HyperFrames design guide.
