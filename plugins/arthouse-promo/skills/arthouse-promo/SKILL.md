---
name: arthouse-promo
description: Recreate or upgrade the arthouse "Top-N open calls this week" promo video — a recurring code-only HyperFrames motion-graphics deliverable in cream-editorial style with high-quality sans type and real event covers. Use when asked to make/regenerate the weekly arthouse open-calls promo, a new week's cut, a quiet/social variant, or to change its format/style. Pulls live calls from the arthouse DB, fetches covers, generates a 12-scene composition, and renders an MP4.
metadata:
  short-description: Weekly arthouse open-calls promo video — format, style, and 4-step pipeline.
  owner: Noam
  created: 2026-05-22
  tags: [arthouse, promo, video, hyperframes, motion, covers, hebrew, open-calls]
  scope: cross-project, cross-tool
---

# Arthouse Weekly Open-Calls Promo

A recurring deliverable: a ~40–46s, 1920×1080 promo of the **top-N live open calls for a given week**,
one call per scene. **Code-only (HyperFrames)** so every call name, deadline, fee and Hebrew line is
accurate — never use an AI video model for the data frames (it hallucinates text). Governed by the
`premium-design-standards` skill; complemented by `motion-graphics-not-slideshow` and `hyperframes`.

## Canonical template (copy / re-run, don't reinvent)

`<local-path>`
is both the proven instance **and** the template. Read its `TEMPLATE.md` for the full playbook and the
4-step pipeline. The generator + scripts there are the source of truth — start from them.

## Format

- 12 scenes: intro → 10 call "deadline-posters" (one call per scene) → outro CTA.
- Each call scene = the event **cover** as hero + index `NN/10`, kicker, title, host, one-line theme,
  AI-policy + fee chips, and the big **deadline numeral + DAYS-LEFT** motif (urgent when ≤7 days).
- 3 rotating layouts (A cover-left/type-right · B mirror · C cover-hero centered) so it never feels repetitive.

## Style (locked — see the template's DESIGN.md)

- **Cream-editorial** identity: paper `#F3EEE4` / card `#FBF8F2` / soft `#EBE4D6`; ink `#1A1612`,
  ink-soft `#4A423A`, muted `#857C70`; scarce accents oxblood `#7A1F1F`, forest `#3F6E44`/`#2F5733`,
  urgent `#B23A0A`. No dark mode, no glow/glass/gradient chrome.
- **High-quality SANS everywhere:** Bricolage Grotesque (Latin) + Heebo (Hebrew), `tabular-nums`;
  two weights carry hierarchy. Hebrew never letter-spaced.
- **Hebrew-first language rule (locked):** ALL labels + the description line are Hebrew — kicker = Hebrew
  location · medium (e.g. `לונדון · צילום · ציור`), chips `מותר AI`/`נדרש AI` + `ללא דמי הגשה`/`דמי הגשה €X`,
  deadline `DD <חודש> YYYY · נותרו N ימים` (`נסגר השבוע` if due this week), optional Hebrew prize line.
  **English ONLY for proper nouns:** event name (title), host, location. Latin numerals/proper-nouns sit in
  `dir="ltr"` runs inside the RTL root. Maps `HE_MONTH`/`HE_LOC`/`HE_MEDIUM` live in `build_promo.py`.
  Don't repeat AI/fee in both kicker and chips (kicker carries location+medium; chips carry AI+fee).
- **Covers:** real photo where available; arthouse generated SVG otherwise; small logos shown `contain`
  on a cream card. Slow Ken-Burns on each cover (never-static).
- **Motion:** entrance-only per scene (mask title reveal, varied eases, 60–90ms staggers); push-slide
  between calls + blur-crossfade bookends; only the outro fades; `transform`/`opacity` only.

## Pipeline (run from the template dir, or a copy for the new week)

```bash
python3 prep_data.py [WEEK_START=YYYY-MM-DD] [N=10]   # arthouse DB -> data/build.json (+ days-left)
python3 fetch_covers.py                                # covers -> capture/assets/ + manifest.json
#   edit the ART curation dict in build_promo.py (per call: variant, short title, host, one-line theme, he:)
python3 build_promo.py                                 # -> index.html
hyperframes lint && hyperframes validate               # 0 errors; contrast clean (text on cream)
hyperframes snapshot --at 1.6,5,12.9,29.2,41.8         # eyeball before the long render
hyperframes render . -f 60 -q high -o renders/<name>.mp4
```

## Notes / gotchas

- `prep_data.py` pool = `status in ('tracking','drafting')`, ordered by soonest deadline; DAYS-LEFT counted from the week-start.
- If a single week has <N live calls, widen by taking the soonest-N from the week-start onward (the default query already does this).
- Prefer a generated SVG cover over a tiny downloaded logo when both exist (crisper, on-brand).
- Run `hyperframes` via the global binary if `npx hyperframes` errors on a missing local package.lock.
- Validate's contrast warnings on stacked hidden scenes are false positives; judge by snapshots/render of the *active* scene.
- Render verified working on this machine; ~3 min for 43s @ 60fps. Then play the MP4 to judge against the `premium-design-standards` quality gate.
