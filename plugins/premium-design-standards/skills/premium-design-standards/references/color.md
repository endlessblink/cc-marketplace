# Color & Surface

The most reliable tell of amateur/AI design is **too many saturated hues at once**.
Premium palettes are controlled, not boring. Restraint is the signal.

## Discipline

- **60-30-10:** 60% dominant (neutrals/background), 30% secondary (type/surfaces), 10% accent (CTAs/focal).
- **1 dominant + 1 accent.** The accent appears on **<10%** of visible surface, is never a large
  fill, and is **never gradient-ified** (a gradient accent reads as indecisive). Two accents fighting = generic.
- AI over-saturates because saturated colors pop in thumbnails. Real editorial/film/luxury work is
  measurably **more desaturated**. Default to lower chroma.

## Ramps — use OKLCH, not HSL

HSL's lightness channel is **not** perceptually uniform — equal numeric steps look unequal.
**OKLCH** (CSS Color 4) fixes this. Build an 11-stop scale (50→950); step L by ~8–10 each;
**taper chroma at both ends** (saturated near-blacks/near-whites look unnatural).

```
oklch(97% 0.01 240)  /* 50  */   oklch(50% 0.15 240)  /* 500 base */
oklch(90% 0.03 240)  /* 100 */   oklch(41% 0.13 240)  /* 600 */
oklch(82% 0.06 240)  /* 200 */   oklch(32% 0.11 240)  /* 700 */
oklch(72% 0.09 240)  /* 300 */   oklch(24% 0.09 240)  /* 800 */
oklch(61% 0.12 240)  /* 400 */   oklch(16% 0.07 240)  /* 900 */
```
Tools: atmos.style, tints.dev, color-ramp.com (OKLCH).

## Near-black / near-white (hard rule)

- **Never `#000000`** for backgrounds — causes halation (glow for astigmatic users) and looks like a
  missing asset on OLED. Use `#0a0a0a` / `#111` / `#1a1a1a`, or warm `#130f0c` / cool `#0d1117`.
- **Never `#ffffff`** for large backgrounds — use `#fafafa`, warm `#f5f5f0`, or cool `#f0f4f8`.
- **Grays have a temperature.** Commit to **one**: warm (hue ~30–60) for editorial/luxury/cinematic,
  cool (hue ~230–260) for tech/minimal. Mixing warm and cool grays = the "muddied" look.

## Premium moves

- **Monochrome + 1 accent** — the most defensible pattern: full gray ramp in one temperature, one chromatic accent for interaction/focal only.
- **Muted/desaturated** — keep chroma <0.08 for backgrounds/supporting colors (e.g. sage + cream + warm-charcoal + one terracotta accent reads expensive).
- **Dark mode done right:** bg `#0f0f11` (not `#000`); text `#e8eaf0` (not pure white — ~14:1, easier on eyes);
  borders `1px solid oklch(100% 0 0 / 0.08)`. No glass-on-flat, no neon glow, no `rgba(255,255,255,0.1)` cards on flat black.
- **Contrast:** WCAG AA 4.5:1 normal / 3:1 large; AAA 7:1 body where achievable.

## The "AI slop" ban list

- **"VibeCode Purple"** `#7c3aed` / violet-500–600 — the single most reliable AI tell.
- **Purple→blue hero gradient** `linear-gradient(135deg,#667eea,#764ba2)`.
- Default Tailwind colors at full saturation as "design" (`bg-blue-500`, `bg-indigo-600`, `bg-purple-500`).
- **Glassmorphism on a flat/solid background** (blur needs something busy to distort).
- **Neon glow** `box-shadow: 0 0 40px rgba(139,92,246,.5)`.
- **Colored left-border cards** `border-left: 3–4px solid <accent>`.
- Stat banners ("10M+ users | 99.9% uptime | 500+ features").
- Centered pill-badge above the H1 ("New: Feature X →").
- Identical 3-up / 4-up feature cards; emoji nav bullets; all-caps section labels without typographic reason.
- **Generic `shadow-md`** `0 4px 6px rgba(0,0,0,.1)`; **universal `rounded-2xl`** on everything.

## Depth & texture (without cheap effects)

- **Layered shadows** (contact + ambient), and the shadow **hue matches the background hue** — gray
  shadows on a colored surface desaturate it and look wrong. Vertical offset ≈ 2× horizontal; opacity drops as blur grows.
```css
--shadow-md:
  0 2px 4px  hsl(220deg 40% 10% / 0.10),
  0 4px 8px  hsl(220deg 40% 10% / 0.10),
  0 8px 16px hsl(220deg 40% 10% / 0.08);
```
- **Grain/noise** at **0.02–0.05 opacity** (SVG `feTurbulence`, `mix-blend-mode: overlay`) — imperceptible but
  reads as "crafted." Above 0.08 it becomes a visible effect.
- **Border XOR shadow, not both.** Border = separation at the same elevation (`1px`, 8–12% opacity of
  foreground). Shadow = elevation change. Both together = unsure.
- **Glass only over busy/image backgrounds**, art-directed: `background: oklch(100% 0 0 / 0.08);
  backdrop-filter: blur(12px) saturate(180%); border: 1px solid oklch(100% 0 0 / 0.12)`. Not blur(40px); not on flat bg; not site-wide.

## Art direction

The generic look = the average of many styles at once. Pick **one** (Editorial / Brutalist / Minimal /
Cinematic-Dark), write "This is [direction] because [reason]", collect 5–8 off-web references.
Commit: ≤2 typefaces, one palette, one radius-per-context, one shadow system + one light direction.

## Color in video / motion

- **Two passes:** correct (match exposure/white balance) → grade (apply a look).
- **Teal-orange** at **~30% intensity**, not 100% (100% reads as a careless LUT). Shadows toward
  teal `hsl(185,60%,30%)`, skin/highlights toward orange `hsl(30,70%,60%)`.
- **S-curve** (lift shadows slightly off 0, pull highlights below 100, add midtone contrast) for a
  film-like toe/shoulder — flat low-contrast frames read as amateur.
- The eye goes to the **highest-chroma element** — use it deliberately; keep motion-graphic overlays
  **desaturated** so they don't fight the footage.
- **Text on video:** near-white `oklch(95% 0 0)` over a semi-opaque dark scrim — not colored text on raw footage.
- **Brand color is punctuation** (lower-thirds, end frames, transition moments, at 60–80% chroma), **never** a section-wide fill.

## Sources
Hype4 60-30-10 · CSS-Tricks (OKLCH, grainy gradients) · Josh W. Comeau (designing shadows) ·
prg.sh "why your AI keeps building the same purple gradient" · DevelopersDigest AI-slop patterns ·
WebAIM contrast checker · ReelMind / PetaPixel (teal-orange) · UX Planet (alternatives to pure black).
