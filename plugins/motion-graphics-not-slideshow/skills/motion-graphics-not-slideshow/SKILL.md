---
name: motion-graphics-not-slideshow
description: Apply when building animated video/motion content (HyperFrames, After Effects-style HTML, Lottie, web animations) that needs to feel like real animation rather than an animated slideshow. Use when the user says the result feels "boring", "static", "like a slideshow", asks for "an actual animation", or says the visual doesn't match what the narration is saying. Provides the seven anti-slideshow techniques, narration-synced storyboarding, and concrete GSAP/CSS patterns for each.
---

# Motion Graphics, Not Slideshow

> **Canonical layer:** this skill is the anti-slideshow checklist. For the broader premium quality bar
> it sits on top of — layout, typography (Hebrew + English), readability/density, color, and the full
> easing/spring/choreography reference — load `premium-design-standards` (`~/.codex/skills/premium-design-standards/`).

If the user is asking for a motion graphic and complaining it's "still a slideshow," they almost certainly mean one of these three things — in order of how often it's the actual problem:

1. **The icon/illustration sits in its corner the whole scene** while wheels spin or a hand rotates in place. Localized fidget is not animation. Animation = actors moving across the frame.
2. **The visual doesn't follow the narration.** The narration says "five middle schoolers stopped a runaway school bus when their driver lost consciousness," but the visual is just a parked bus icon. The visual must illustrate the narration phrase by phrase.
3. **Every text element is frozen** after its entry animation. Headlines appear and sit. Counters count up and stop. Labels never move again.

If you build the entire piece with techniques 1–6 below but skip technique 7 (narration-synced choreography of moving actors), the user will still call it a slideshow, and they'll be right.

**Quick self-test.** Pause any random frame. Does it look like a clean, settled poster? Slideshow. Does it look blurred / mid-transition / in-progress / with elements crossing the frame? Motion graphics.

## The Seven Anti-Slideshow Techniques

### 1. Continuous camera motion within every scene

Every scene's whole layout slowly drifts, zooms, or pans across its lifetime — never frozen.

```js
var sceneDur = SCENES[idx + 1].start - SCENES[idx].start;
tl.fromTo(s + " .scene-content",
  { scale: 1.00, x: 0, y: 0 },
  { scale: 1.06, x: -25, y: -8, duration: sceneDur, ease: "none" },
  SCENES[idx].start);
```

Use `ease: "none"` for true camera linearity. Vary direction per scene so it doesn't feel mechanical.

### 2. Whip-pan transitions with motion blur

Outgoing scene slams off-screen with horizontal blur; incoming slams in from the opposite side with blur clearing.

```js
function whipPan(outId, inId, t) {
  tl.to(outId, { x: -300, filter: "blur(20px)", opacity: 0, duration: 0.30, ease: "power4.in" }, t);
  tl.fromTo(inId,
    { x: 300, filter: "blur(20px)", opacity: 0 },
    { x: 0, filter: "blur(0px)", opacity: 1, duration: 0.40, ease: "power3.out" },
    t + 0.20);
}
```

### 3. Kinetic typography at the character level

Split words into characters; per-char stagger entry with rotation/slide.

```js
function splitChars(el) {
  el.querySelectorAll('.word').forEach(function (word) {
    var text = word.textContent;
    word.textContent = '';
    text.split('').forEach(function (ch) {
      var c = document.createElement('span');
      c.className = 'char';
      c.textContent = ch;
      c.style.display = 'inline-block';
      word.appendChild(c);
    });
  });
}

tl.from(s + ' .char',
  { y: 80, rotation: 8, opacity: 0, duration: 0.50, stagger: 0.018, ease: "back.out(1.4)" },
  t + 0.45);
```

### 4. Counter animations on every meaningful number

Static numbers feel like text. Counters create "happening now" energy.

```js
function counter(el, from, to, dur, t, format) {
  var obj = { v: from };
  tl.to(obj, {
    v: to, duration: dur, ease: "power2.out",
    onUpdate: function () { el.textContent = format(obj.v); }
  }, t);
}
```

### 5. Persistent elements crossing scene boundaries

A scrolling ticker, progress bar, or background pattern that runs through the ENTIRE composition without resetting per scene.

```js
tl.fromTo("#ticker-inner",
  { x: 0 }, { x: -2000, duration: TOTAL_DURATION, ease: "none" }, 0);
```

### 6. Grain / animated noise overlay

Subtle drifting SVG turbulence at 5–10% opacity kills the flat digital look.

```html
<svg class="grain"><filter id="g"><feTurbulence type="fractalNoise" baseFrequency="0.9"/></filter><rect width="100%" height="100%" filter="url(#g)"/></svg>
```

---

### 7. (THE BIG ONE) Narration-synced animated storyboards — actors enter, move across, and exit

This is the technique that separates real animation from "decorated slideshow." Most users complaining the result is a slideshow are missing this, even when 1–6 are all in place.

**The principle.** Each scene is a 6–9-second mini-narrative driven by the narration. Multiple SVG **actors** share the stage. Each actor:
- **Enters** at a specific narration phrase (slides in from off-stage, drops in from above, scales up from a point)
- **Moves across** the stage during its on-screen window (translation, transformation, looped sub-behavior)
- **Exits** when the narration moves on (slides off, fades, gets replaced by the next actor)
- **Coexists** with other actors when their windows overlap (a bus with kids visible inside while Z's float over the driver)

A single icon that "appears, sits, and fades" is a slideshow card. A choreographed sequence of multiple actors entering and exiting in sync with the narration is animation.

**Wrong (the "v4 trap").** A bus icon parked in the corner of every frame for 8.5 seconds while wheels spin in place. Headline narration says "five middle schoolers stopped a runaway school bus when the driver lost consciousness" — but the visual shows only a bus.

**Right.** Storyboard for the same scene:

| t (rel) | Actor | Motion |
| --- | --- | --- |
| 0.3 | road-horizon | draws in left-to-right |
| 0.5 | "MISSISSIPPI" pin | drops in from top, settles, fades at 2.0 |
| 2.0 | five-students cluster | pops up from below |
| 3.5 | bus | drives in from right (x: +600 → 0), wheels spinning |
| 4.5 | sleep-Z's | float up over driver position |
| 5.5 | bus brakes | sudden stop + skid lines |
| 6.0 | hero student | breaks from cluster, slides to bus front |
| 6.5 | lightbulb | bursts in |
| 7.5 | HEROES badge | rotates in |

Multiple actors visible at any single moment. Things actually move across the frame.

**Implementation pattern in HyperFrames + GSAP:**

```html
<!-- Per-scene HTML: a stage with multiple actors -->
<div class="story-stage" data-scene="1">
  <div class="actor" id="a-1-road"><svg>...</svg></div>
  <div class="actor" id="a-1-pin"><svg>...</svg></div>
  <div class="actor" id="a-1-students"><svg>...</svg></div>
  <div class="actor" id="a-1-bus"><svg>...</svg></div>
  <div class="actor" id="a-1-zs"><svg>...</svg></div>
  <div class="actor" id="a-1-light"><svg>...</svg></div>
  <div class="actor" id="a-1-hero"><svg>...</svg></div>
</div>
```

```css
.story-stage { position: absolute; top: 240px; right: 100px; width: 580px; height: 540px; }
.actor { position: absolute; inset: 0; opacity: 0; will-change: transform, opacity; }
```

```js
// STORYBOARDS data — one cue list per scene
var STORYBOARDS = {
  1: [
    { id: 'a-1-road',     enter: 0.3, exit: 8.4, from: { scaleX: 0 },                to: { scaleX: 1, opacity: 1 },     ease: 'power3.out',  dur: 0.5 },
    { id: 'a-1-pin',      enter: 0.5, exit: 2.0, from: { y: -200, opacity: 0 },      to: { y: 0, opacity: 1 },          ease: 'back.out(1.6)', dur: 0.5 },
    { id: 'a-1-students', enter: 2.0, exit: 8.4, from: { y: 400, opacity: 0 },       to: { y: 0, opacity: 1 },          ease: 'back.out(1.4)', dur: 0.5 },
    { id: 'a-1-bus',      enter: 3.5, exit: 8.4, from: { x: 700, opacity: 0 },       to: { x: 0, opacity: 1 },          ease: 'power3.out',  dur: 1.2, behavior: 'wheels' },
    { id: 'a-1-zs',       enter: 4.5, exit: 6.0, from: { y: 30, opacity: 0 },        to: { y: -40, opacity: 1 },        ease: 'sine.inOut',  dur: 1.0 },
    { id: 'a-1-light',    enter: 6.5, exit: 7.5, from: { scale: 0, opacity: 0 },     to: { scale: 1, opacity: 1 },      ease: 'back.out(2.0)', dur: 0.4 },
    { id: 'a-1-hero',     enter: 7.5, exit: 8.4, from: { scale: 0, rotation: -180 }, to: { scale: 1, rotation: 0, opacity: 1 }, ease: 'back.out(1.6)', dur: 0.6 }
  ],
  // ...one for each scene
};

function playStoryboard(idx, sceneStart) {
  STORYBOARDS[idx].forEach(function (cue) {
    var sel = '#' + cue.id;
    // Entry
    tl.fromTo(sel, cue.from, Object.assign({}, cue.to, { duration: cue.dur, ease: cue.ease }),
      sceneStart + cue.enter);
    // Exit (fade + small scale shift)
    tl.to(sel, { opacity: 0, scale: 0.92, duration: 0.30, ease: 'power2.in' },
      sceneStart + cue.exit);
    // Optional in-place behavior while on stage (wheels spin, eye blink, etc.)
    if (cue.behavior) applyBehavior(cue.behavior, sel, sceneStart + cue.enter, cue.exit - cue.enter);
  });
}
```

**Designing a storyboard for a new scene** (do this BEFORE writing SVGs or code):

1. **Read the narration aloud.** Identify the 4–7 distinct phrases.
2. **For each phrase, name an actor** that visualizes it. Concrete nouns and verbs become actors. Adjectives become looped behaviors on existing actors.
3. **Decide entry direction.** Locations come from off-stage (state shape slides in). Numbers drop from above. Heroes/badges scale-burst from center. Subjects of the story (people, animals, objects) drive in from edges.
4. **Decide exit.** First actors usually fade or exit toward the corner where the next actor enters. Climax actors stay on stage until scene end.
5. **Overlap.** At least 2 actors should be visible together for ~30% of the scene's runtime. That's what makes the scene feel layered/animated rather than card-by-card.
6. **Looped sub-behavior.** Bus has wheels spinning while it drives. Eye iris dilates while the eye is on stage. Heart pulses while it's visible.

**Frame-level self-test for narration sync:** for every distinct narration phrase, pause the render at the midpoint of that phrase. The actor that names the noun in that phrase MUST be visible. If the narration says "driver lost consciousness" and the only thing on screen is a still bus, the storyboard is wrong.

**Common storyboard archetypes** (reuse these across stories):
- **Location intro** (state/country shape drops or slides in) → fades by phrase 2
- **Subject ensemble** (multiple small figures pop up) → stays as backdrop
- **Vehicle/object motion** (bus drives, ship sails, runner crosses) → moves across stage horizontally
- **Status change** ("OPEN" → "CLOSED" sign flips, eye closed → eye opens) → in-place transformation
- **Build-up sequence** (calendar pages flip, coins rain down, sprouts grow) → many small actors stagger-spawn
- **Climax stamp** (hero badge, trophy, "FIRST" stamp) → scale-bursts in at end

---

## Layout Variation Across Scenes

Identical templates = slideshow. Vary at minimum 2–3 scenes:
- Full-frame illustration with text overlay
- Center-stage typographic moment, no illustration
- Diagonal split
- Bottom-third text bar with illustration filling top 2/3

Milestone scenes (5/10, intro, outro) should break the template.

## Pacing Variation

Slideshows have uniform timing. Motion graphics have rhythm:
- **Quick hits** (3–5s): rapid facts
- **Medium dwell** (6–8s): standard
- **Slow burn** (10–12s): emotional/complex
- **Beat moments** (1–2s): pure typography or single-image flashes

## Anti-patterns to actively avoid

- **One icon per scene that "performs in place"** — wheels spin, eye blinks, hand rotates, but the icon never crosses the frame. This is the #1 failure mode and the most common reason users say "still a slideshow." Use technique 7 instead.
- **Element fades in, sits, fades out** with only a localized loop — once entered, an actor must either MOVE (translate) or eventually get REPLACED by another actor entering for the next narration phrase.
- **Identical eases everywhere** — vary across at least 4 different eases per scene.
- **Crossfade between scenes** — fine for warm/calm only; otherwise directional motion.
- **Centering everything** — diagonal energy, off-center compositions, intentional asymmetry.
- **Visual disconnected from narration** — if the narration introduces a new noun every 1.5s but the visual stays the same, you have a slideshow with audio.

## Quick Self-Test

Three pass/fail checks. If any fails, you have a slideshow:

1. **Frozen-frame test.** Pause at any random second. The frame should look in-progress, not poster-clean. At least 30% of pixels in motion (>10 px/s).
2. **Narration-sync test.** Pause at the midpoint of each distinct narration phrase. The actor that visualizes that phrase's noun must be visible. If the narration says "driver lost consciousness" and the screen shows only a bus, fail.
3. **Cross-frame test.** Within any 2-second window, at least one actor should ENTER or EXIT or move at least 100px across the stage. If everything is just bobbing in place for 2s, fail.

## HyperFrames-Specific Notes

- Build continuous-motion tweens with `ease: "none"` for camera-style linearity.
- Calculate finite repeats — `repeat: -1` is banned; use `Math.ceil(totalDur / cycleDur) - 1` or odd values for proper yoyo termination.
- Char/word splitting must run synchronously before timeline construction.
- Whip-pans: 0.30–0.45s, with 60–80ms overlap between out and in.
- Persistent elements (ticker, grain, progress) live at the root composition level.
- **SVG group rotations:** use GSAP's `svgOrigin: "x y"` (raw SVG coords, no `px`), not CSS `transformOrigin`. With `transformOrigin` GSAP computes the pivot from the HTML bounding box, which is wrong for `<g>` elements inside an SVG — the rotation will fly the actor off the stage. This is a silent failure mode that ate a full render cycle on a real project.
- Actors should be `<div class="actor">` wrappers with inline SVG inside, not raw `<g>` elements at the root — that gives GSAP a CSS-positionable handle for the entrance/exit translations while still letting it dive in for in-place behaviors.

## Workflow when the user says "still a slideshow"

1. **Don't add more loops.** Adding bobbing, breathing, or wave behaviors to a static layout will not solve it.
2. **Audit narration sync first.** For each scene, list the narration phrases and ask: is there a distinct visual actor entering/exiting for each phrase? If no, this is the problem.
3. **Storyboard before coding.** Write the actor table (phrase → actor → entry direction → exit) on paper first.
4. **Test scene 1 in isolation.** Render only the first scene's window before fanning out — the storyboard architecture either works or it doesn't, and you'll know in one render.
5. **Watch with audio.** A render with no audio still looks like a slideshow if the storyboard isn't synced. Always preview with narration to verify the actor enters at the right phrase.
