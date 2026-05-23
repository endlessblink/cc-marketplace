# Readability & Content Density

The most common failure: **too much text, too much on screen.** It's not a style problem,
it's a cognitive-load problem. Premium work shows **less** and trusts what remains.

## Text on screen in video — hard limits

| Content type | Max words | Max lines |
|---|---|---|
| Motion-graphic frame (silent) | 15 | 3 |
| Caption / subtitle block | 7–10 | 2 |
| Title card / lower third | 6–8 | 1–2 |
| Kinetic-type beat | 3–7 | 1 |

- **≤30 characters per line** on a video text line (hard limit).
- Frames approaching **30 words must be split** into two frames.
- **"If it takes a paragraph, it's not a video frame."** It belongs in a document.
- The 6×6 rule (6 bullets × 6 words) is a *ceiling*, not a target — for video it's already too generous.

## Reading speed & dwell time

- Screen reading: **200–250 wpm** (10–30% slower than print). Design for **150–180 wpm** for mixed audiences.
- **Kinetic-type sweet spot: ~90 wpm** (Dolic et al., GRID-2024) — viewers process animation + text together.
- On-screen text minimum: **1 second per 13 characters**, **+25–50%** on busy/animated backgrounds,
  **+50–100%** for dense data, numbers, URLs, legal copy.
- **Never display any text for < 1.25s** — below that, viewers scan fragments, not meaning.
- Reading time starts when text is **stationary**, not when its entrance animation begins.

| Text volume | Min display time |
|---|---|
| 1–3 words / label | 2–3s |
| short sentence (4–8 words) | 3–5s |
| medium (8–12 words) | 4–6s |
| long (13–20 words) | 6–8s |

## Readability metrics

- **Line length:** body 50–75 chars, ideal ~66 (Bringhurst); video line ≤30 chars; mobile 30–50.
  Below ~45 breaks rhythm; above ~75 the eye loses the next line's start.
- **Line-height:** 1.5 body (WCAG 1.4.8 floor), 1.6–1.7 for long lines, 1.2–1.35 OK for short display lines.
- **Min font size:** 16px web body · 40–60px video body · 60–90px video title (title ≥1.5× body).
- **Contrast (WCAG AA):** 4.5:1 normal, 3:1 large (≥24px or bold ≥18.5px); AAA 7:1 for body if reachable.
  In video, black/white is max contrast; avoid red-green; use a dark scrim or 2–4px outline over footage.
- **Hebrew:** size at **115–125% of the Latin size** for visual parity; ensure leading clears niqqud.

## Cognitive load

- Design for **~4 chunks** (Cowan), not Miller's 7±2. Don't use "7±2" to justify a 7-item menu.
- **≤4–5 active elements** requiring attention on a screen at once. A crowded screen communicates *less*.
- **Whitespace measurably improves comprehension** (and perceived value). Removing an element increases the impact of those that remain.
- **Progressive disclosure:** reveal in stages; in video, animate elements on **one at a time**, not all at once.

## Hierarchy for readability

- **One focal point per screen.** If the viewer must choose where to look, you've spent their attention on navigation.
- Reading patterns: **F-pattern** on text-heavy screens, **Z-pattern** on sparse ones → put the key
  content **top-left / top strip**, never buried at the bottom.
- **Cut text ruthlessly:** drop any word whose removal doesn't change meaning; prefer a visual/diagram
  over prose; keep nouns + verbs, kill adjectives/adverbs; if it needs >2 lines on a frame, split it.

## Video — voice vs. on-screen text (Mayer's multimedia learning)

- **Redundancy effect:** narration + identical on-screen text → *worse* retention than narration alone.
  **Do not put your script on screen.** If the voice says it, the screen shows a visual, one keyword, or nothing.
- **Split-attention effect:** if text and a visual must be mentally integrated, place the label
  **adjacent** to what it describes — or drop it and let the voice carry it.
- **On-screen design text ≤ 10–15% of the information.** Voice carries the rest.
- **Captions ≠ design text.** Captions = speech transcription (≤2 lines, 32–45 chars, lower third, scrim,
  18–24pt). Never run voiceover + caption + design text simultaneously — collapse to ≤2 reading tasks.

## Anti-clutter bans (build into any template)

- MAX 15 words / video frame, MAX 3 lines, MAX 30 chars/line, MAX 2 caption lines.
- MIN 40px body @1080p, MIN 16px web body, MIN 4.5:1 contrast.
- MAX 4–5 simultaneous active elements; one focal point.
- Never < 1.25s on screen; honor 1s / 13 chars + buffer.
- Never mirror the narration as on-screen text.
- Every element must answer "why is this here?" — if it can't, remove it.

## Sources
legibility.info · Baymard / UXPin (line length) · WCAG 2.1 SC 1.4.3 & 1.4.8 · Mayer, *Cognitive
Theory of Multimedia Learning* (redundancy, split-attention) · Laws of UX (Miller / Cowan) ·
NN/g F-pattern · Dolic et al., temporal typography, GRID-2024.
