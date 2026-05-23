# Motion

What separates premium motion (Linear, Vercel, Apple, Stripe, film titles) from cheap/slideshow
motion is **intentional easing, distance-aware timing, overlap, and never-static frames.**
This file is the *why/what*; for API specifics defer to `gsap` and `remotion-best-practices`.

## Easing — the core quality bar

- **No `linear` on visible motion.** Only acceptable for looping spinners and scroll-scrubbed transforms.
- Built-in CSS easing is too weak — prefer custom cubic-bezier.
- For UI, **ease-out for both entrances and exits** (Emil Kowalski) — front-loaded acceleration reads as responsive.

**Named premium curves:**

| Name | cubic-bezier | Use |
|---|---|---|
| easeOutExpo | `0.19, 1, 0.22, 1` | **workhorse** — large elements entering |
| easeOutQuint | `0.23, 1, 0.32, 1` | hero text, modal entrance |
| easeOutQuart | `0.165, 0.84, 0.44, 1` | standard UI reveals |
| Vercel-smooth | `0.16, 1, 0.3, 1` | page transitions, drawers |
| easeInOutQuint | `0.86, 0, 0.07, 1` | repositioning across screen |
| back / overshoot | `0.34, 1.56, 0.64, 1` | scale-in, success, hover — **spring-feel from pure CSS** |
| anticipation | `0.68, -0.55, 0.27, 1.55` | "launched/thrown" elements (negative y1 = pull-back) |
| Material standard | `0.4, 0, 0.2, 1` | state toggles, icon transitions |

Overshoot mechanic: a y2 > 1 makes a `transform` briefly exceed its target before settling — spring
feel without JS. On `opacity` it's clamped (invisible) — don't bother.

**Decision tree:** enter off-screen → expo/quint · reposition on-screen → inOutQuint · exit → faster
easeIn · micro-interaction → spring or back · opacity/color only → easeOut · loop → linear only.

## Springs — real numbers

- UI feel: **stiffness 200–300, damping 22–28, mass 0.8–1.2.**
- **Apple-premium:** `duration 0.5s, bounce 0.2` (critically damped + slight overshoot). bounce >0.5 = toy-grade.
- react-spring presets: default 170/26 · gentle 120/14 · stiff 210/20 · wobbly 180/12.
- **Spring vs tween:** springs for interruptible / position / scale / semantically-weighty motion;
  tweens for opacity & color (overshoot invisible) and high-frequency micro-interactions.
- **Never force `duration` on a spring** — it kills the physics that make it worth using. Use
  `visualDuration` (Motion) or, in Remotion, `spring({ config, durationInFrames })` to lock to the beat grid.

## Premium techniques (with numbers)

- **Staggered reveals:** list 40–60ms · cards 60–80ms (`y:20, scale:0.97, opacity:0`) · hero text lines 60–80ms.
  **Cap the total stagger sequence at ≤400ms** — if 8 items × 80ms = 640ms, the last is too late (reduce stagger or cap visible items).
- **Shared-element transitions:** `layoutId` + spring `stiffness:300, damping:30` (the Vercel active-tab pill).
- **Magnetic hover:** map mouse delta × 0.2–0.4, spring back on leave.
- **Scroll choreography:** parallax ratios 0.2 / 0.5 / 0.7 / 1.0× (bg→fg, max 4–5 layers); GSAP `scrub: 0.5–1.0`
  with `ease:"none"` (the scrub lag *is* the smoothing — don't double up).
- **Micro-interactions:** button press 80–100ms `scale:0.97` · hover 150–200ms · modal 250–300ms.

## UI durations

`<300ms` for UI micro-motion (180ms reads more responsive than 400ms); drawers/large surfaces ≤500ms.
Duration ladder: 100 / 150 / 200 / 300 / 500ms. Start entrances from `scale: 0.9–0.93`, **never 0**;
make `transform-origin` match the trigger location.

## Video motion

- **Never-static rule:** ≥1 layer is always moving. When content has "landed," start ambient motion —
  Ken Burns `scale 1 → 1.06–1.08` over the scene, parallax drift, breathing `1 → 1.02`, or a grain overlay.
  A static hold after entrance = slideshow.
- **Kinetic typography — premium vs cheap:**
  - *Cheap:* whole block fade in/out; all words at once; letter-spacing expansion as the reveal; per-letter scale-from-0.
  - *Premium:* **mask/clip reveal** — text in `overflow:hidden`, translate `y:110% → 0`, GSAP SplitText
    `{ type:"lines", mask:"lines" }`, stagger 60–90ms, `power3/power4.out`, 0.4–0.7s/line.
  - *Editorial:* word stagger `y:12px, opacity:0→1`, 40ms, 350ms, `power2.out` — most readable in fast sequences.
  - *Numbers:* count up by driving a spring to the destination value (feels earned vs a linear counter).
- **Transitions beyond cut/fade:** match cut · whip pan (blur 20–30px over 4–6 frames, resolve in incoming
  shot) · clip-path/mask morph (`circle(0%)→circle(150%)`, 400–600ms) · push/slide (`power3.inOut`, 300–400ms)
  · crossfade-with-hold (overlap 4–8 frames so there's no "hole in space").
- **Choreography in temporal layers:** ground (always moving) · content (enters mid-scene, settles) ·
  accent (enters after content settles, staggered) · exit (overlapping). **Overlap entrances and exits** —
  GSAP `-=0.3` position offset; element B starts while A is still mid-exit. Overlap ≈ 30–50% of the prior duration.
- **Audio sync:** Remotion `useAudioData` + `visualizeAudio` to drive scale/opacity from bass amplitude, or
  convert beat timestamps to frames and start a `spring({ frame: frame - beatFrame, fps })` on each beat.

## Anti-cheap bans

- No `linear` on visible motion (loops/scrub excepted).
- No **uniform duration** regardless of distance — scale roughly with `sqrt(distance)`; small elements snap faster.
- No **simultaneous entrances** — minimum 30ms stagger even for things that "appear together."
- No **fade-only reveals** — always pair opacity with a transform (even `y: 8px`).
- **Exits ≠ reversal of entrance** — exits are faster (users don't need to read something leaving) and ideally a different direction/ease.
- No forced-`duration` springs.
- **No static video frames.**
- No `scale: 0 → 1` without an ease (spring or back).
- **Animate `transform` + `opacity` only** — never `width/height/padding/margin/top/left` (layout/reflow/repaint → dropped frames).

## Framework bridges

**GSAP** — relative timeline positioning gives the "movie-trailer" feel:
```js
const tl = gsap.timeline();
tl.from(".headline", { y: 48, opacity: 0, duration: 0.6, ease: "power3.out" })
  .from(".subhead",  { y: 24, opacity: 0, duration: 0.5, ease: "power2.out" }, "-=0.3")
  .from(".cta",      { scale: 0.92, opacity: 0, duration: 0.4, ease: "back.out(1.4)" }, "-=0.2");
// SplitText (free in 3.13+): new SplitText(el,{type:"lines",mask:"lines"}); gsap.from(lines,{y:"110%",stagger:0.08,ease:"power4.out"})
// ScrollTrigger scrubbed: { scrub: 0.8 } + ease:"none"
```

**Remotion** — always clamp; manual stagger; lock springs to frames:
```tsx
const y = interpolate(frame, [0, 18], [24, 0], { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' });
const o = spring({ frame: Math.max(0, frame - i * 5), fps, config: { stiffness: 300, damping: 24 } });
// never-static bg: interpolate(frame, [0, durationInFrames], [1, 1.06], { extrapolateRight: 'clamp' })
```

## Quick reference

```
Spring feel:        stiffness 200–300, damping 22–28, mass 0.8–1.2
Apple premium:      duration 0.5s, bounce 0.2
List/card stagger:  40–80ms, cap sequence ≤400ms
Text mask reveal:   y:110%→0, stagger 60–90ms, power3/4.out
Micro:              press 80–100ms, hover 150–200ms, modal 250–300ms
Workhorse easing:   cubic-bezier(0.19, 1, 0.22, 1)  (easeOutExpo)
Overshoot (CSS):    cubic-bezier(0.34, 1.56, 0.64, 1)  (transforms only)
Scrub lag:          0.5–1.0s
Parallax:           0.2 / 0.5 / 0.7 / 1.0×
Never-static:       ambient scale 1 → 1.06–1.08 over full scene
```

## Sources
react-spring / Motion / Remotion docs · Apple WWDC23 "Animate with springs" · Emil Kowalski
"7 Practical Animation Tips" · reubence "The Easing Blueprint" · uxderrick web-animation gist ·
GSAP ScrollTrigger / SplitText docs · Vercel-tabs (joshuawootonn, frontend.fyi) · Disney's 12 principles.
