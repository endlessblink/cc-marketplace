---
name: premium-design-standards
description: Canonical, opinionated quality bar for premium design AND motion across video and UI — layout/composition, typography (Hebrew + English / bidi), readability & content density, color & surface, and motion/animation. Tech-agnostic principles with concrete numbers, named font/curve/color choices, and explicit BAN lists for the generic "AI slop" look. Load before designing or choreographing any asset (Remotion, HyperFrames, GSAP, web/app UI). Primary consumer is Showrunner's programmatic video pipeline. Complements gsap/remotion-best-practices (the how) and motion-graphics-not-slideshow (anti-patterns).
metadata:
  short-description: Premium design + motion standards — layout, type (He/En), readability, color, motion. Opinionated, with ban lists.
  owner: Noam
  created: 2026-05-21
  tags: [design, motion, layout, typography, hebrew, rtl, color, readability, composition, video, anti-slop]
  scope: cross-project, cross-tool
---

# Premium Design Standards

The quality bar for any visual asset — video frame, motion sequence, or UI screen.
This is **taste made concrete**: numbers, named choices, and ban lists, not vibes.
The goal is to stop producing generic / "AI slop" output and hit a premium bar on
**every** axis: layout, typography (Hebrew + English), readability, color, and motion.

## When to use

Load this **before** designing or choreographing anything — a title card, a scene,
a landing page, a dashboard, a kinetic-type sequence. If you're about to pick a font,
a color, a spacing value, an easing curve, or a layout, this skill governs the choice.

**Two registers — declare which you're in before you start. Rules differ:**
- **VIDEO / motion-graphics** (Showrunner — Remotion, HyperFrames): slower, expressive,
  story-paced; safe areas; never-static frames; legibility at distance; voice + visuals.
- **UI** (web/app): fast (<300ms), functional, responsive; interaction states; density.

A rule that's right for one is often wrong for the other — each reference file marks
register-specific guidance.

## What's where (read the reference for depth)

This file is the **index + the quality gate**. Each domain has a dense reference file
with the actual numbers, tables, and ban lists. Pull the relevant one(s) before working:

| Domain | File | Use when |
|---|---|---|
| Layout & composition | `references/layout.md` | placing anything: grids, hierarchy, framing, spacing |
| Typography (He + En) | `references/typography.md` | picking fonts, type scale, RTL/bidi, on-screen text size |
| Readability & density | `references/readability.md` | how much text/content fits, dwell time, voice-vs-text |
| Color & surface | `references/color.md` | palettes, dark mode, shadows, the AI-slop ban list, grading |
| Motion | `references/motion.md` | easing, springs, choreography, transitions, kinetic type |

Complementary skills (don't duplicate them — defer):
- `gsap`, `remotion-best-practices` — the framework **how** (API). This skill is the **why/what**.
- `motion-graphics-not-slideshow` — the full anti-slideshow checklist for video.
- `[[noam-personal-preferences]]` — operator-specific corrections (visual-weight↔feature-weight,
  3–5s/shot, no dead time). They override defaults here when they conflict.
- `[[rtl-design]]`, `[[hebrew-content-qa]]` — RTL implementation + Hebrew copy QA.

## Step 0 — Pick a direction first (the anti-slop foundation)

The generic look is **the average of many styles applied at once** — some brutalist type,
some glass, some bento, some editorial. Averaging produces nothing. Before any pixels:

1. Commit to **one** direction: **Editorial · Brutalist · Minimal · Cinematic-Dark**.
2. Write one sentence: *"This is [direction] because [reason]."*
3. Gather 5–8 references from **outside** web/UI (film stills, book covers, magazine spreads,
   architecture). That set is your visual language.
4. Evaluate every later decision against the sentence. If an element doesn't belong to the
   chosen direction, cut it.

Commit to: **max 2 typefaces · one palette · one radius-per-context · one shadow system + one
light direction.** Consistency is the cheapest premium signal there is.

## The Quality Gate (pre-ship checklist)

Run this before calling any asset done. Each line is a hard rule; the reference file has the why.

**Layout** — `references/layout.md`
- [ ] All spacing on the 8pt scale `{8,16,24,32,48,64,96,128}` (4pt only for micro). No 10/12/15/20/25/30.
- [ ] One clear focal point. Hierarchy survives a 10px blur (squint test).
- [ ] Not centered-everything — asymmetry unless symmetry is semantically justified.
- [ ] ≥40% empty on hero/title scenes; nothing touches frame edges.
- [ ] (Video) all text inside title-safe (≥96px L/R, 54px T/B at 1080p).

**Typography** — `references/typography.md`
- [ ] No banned faces (Latin: Inter-only/Roboto/Poppins/Lato/Open Sans/Montserrat; Hebrew: Heebo/Rubik/Assistant/David).
- [ ] ≤2 typefaces; 2 weights doing the work (400 + 700/900), size carries hierarchy.
- [ ] Hebrew: never positive letter-spacing; line-height 1.5–1.6 body; `dir="rtl"` set; inline LTR wrapped in `<bdi>`.
- [ ] (Video) body ≥40px, title 72–120px at 1080p.

**Readability** — `references/readability.md`
- [ ] (Video) ≤15 words/frame, ≤3 lines, ≤30 chars/line; on screen ≥1.25s and ≥1s per 13 chars.
- [ ] ≤4–5 active elements on screen at once.
- [ ] Narration script is **not** mirrored as on-screen text (redundancy effect).
- [ ] Body line length 50–75 chars; line-height ≥1.5; contrast ≥4.5:1 (3:1 large).

**Color** — `references/color.md`
- [ ] 1 dominant + 1 accent; accent on <10% of surface, never a large fill or gradient.
- [ ] No `#000` / `#fff` backgrounds (near-black / near-white instead); grays one temperature.
- [ ] No AI-slop tells: VibeCode-purple, purple→blue hero gradient, glass-on-flat, neon glow,
      left-border cards, default `shadow-md`, universal `rounded-2xl`, emoji nav, stat banners.
- [ ] Shadows layered with hue matching the background; grain (if any) at 0.02–0.05 opacity.

**Motion** — `references/motion.md`
- [ ] No `linear` on visible motion (loops/scrub only); no fade-only reveals (always pair a transform).
- [ ] Animate `transform`/`opacity` only — never width/height/top/left/margin.
- [ ] Duration scales with distance; entrances stagger (≥30ms); exits faster than entrances.
- [ ] (Video) no static frame — at least one layer always moving; text held static while readable.
- [ ] Easing is intentional (e.g. easeOutExpo `cubic-bezier(0.19,1,0.22,1)` for entrances), not default.

If any box is unchecked, it's not done. When in doubt, **remove** — premium shows less and trusts what remains.
