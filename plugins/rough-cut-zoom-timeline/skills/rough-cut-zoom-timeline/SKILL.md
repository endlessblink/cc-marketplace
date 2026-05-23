---
name: rough-cut-zoom-timeline
description: Use when working on Rough Cut's single shared timeline across Recording edit and NLE toolsets, clip trim/drag/split behavior, shared sync, zoom markers, Timeline panel UX, or preview/export parity. Applies to rough-cut-mvp.
---

# Rough Cut Zoom Timeline

Use this skill for `<local-path>` when changing the one shared timeline used by both Recording edit and NLE, including zoom markers, cuts, clips/tracks, trim/drag/split interactions, or preview/export transforms.

## Product Rules

- Timeline panel should be compact and pro-editor-like: no hero cards, no huge buttons, no paragraph-heavy help text.
- Avoid horizontal overflow in the left setup board. Use compact rows, `min-width: 0`, text truncation, and scoped overflow guards.
- Inspector zoom controls should be stable: sliders are present consistently and disabled when no zoom segment is selected, instead of appearing suddenly and making the panel jump.
- Zoom segments must be draggable and resizable directly in the timeline.
- Overlapping zoom segments are allowed. When multiple zoom markers are active, the longest duration marker takes precedence; tie-break deterministically by earlier start frame, then id.
- Overlapping zooms should render on separate visual layers in the timeline so both segments remain discoverable.
- Persist marker edits on drag end, not on every pointer move.

## Shared Timeline Architecture Rules

- There is one timeline. Recording edit and NLE are different tools/views over that same timeline, not separate timelines and not a primary/derived hierarchy.
- Recording edit and NLE are both canonical editing surfaces. Do not treat either one as a read-only projection, second-class collapsed view, or forked edit model.
- The source of truth is the shared project timeline model, not either tab's component-local state. Both tabs must read/write that same model through the app's project-change path.
- Recording edit may present simplified tools for the primary screen-recording workflow, but its cuts, trims, zooms, cursor/click effects, camera PiP, aspect, background, and export settings must be edits on the same timeline the NLE uses.
- NLE may present multi-track detail and advanced tools, but it must preserve Recording edit semantics for screen-recording workflows. Switching tabs must not silently reinterpret, drop, flatten, or fork edits.
- Before adding trim, drag, split, generated assets, or export behavior, define the shared timeline invariant and the tool-specific interaction rule for both surfaces.
- Existing `cutRanges`, trim state, zoom markers, camera/cursor presentation, and export settings are persisted user edits. Migrations must preserve current export output unless a task explicitly changes behavior.

## Professional Playback Architecture Rules

- Canonical timeline time is the single source of truth for preview playback, scrubbing, jog/shuttle, export parity, and active-clip resolution.
- Playback must be wall-clock / virtual-timeline driven with one playhead clock, usually advanced by `requestAnimationFrame` from elapsed `performance.now()` time and playback rate.
- Hidden `<video>` elements are decode surfaces only. They must not own the clock, and `HTMLVideoElement.currentTime` must not be treated as canonical timeline time.
- Resolve active media from canonical timeline time: `localTime = sourceIn + (timelineTime - timelineIn)` for clips whose half-open range contains the playhead.
- Timeline gaps are first-class. A gap resolves to no active clip and should render black/empty while the virtual playhead continues moving.
- Source trims and hidden-start clips must be handled by resolver math, not by trying to make a media element's natural playback timeline line up with composition time.
- Scrubbing should set canonical timeline time directly and render that frame; it should not depend on video element playback side effects.
- Future multi-track compositing, transitions, cursor/click overlays, camera PiP, audio scheduling, and export/EDL logic must sample from the same canonical timeline clock.
- Use decoder pooling/preload/pre-seek for smoothness, but keep the pool subordinate to the timeline clock. Avoid creating one permanent video element per clip unless there is a measured need.
- When uncertain, prefer tests that prove the virtual playhead advances through gaps, source offsets, and clip transitions independent of native video `timeupdate` events.

## Professional Timeline Interaction Rules

- Edge trims and clip drags must use a local interaction session/preview state during pointer movement. Commit one pure project mutation on pointerup for undo/redo.
- Do not mutate the project on every pointermove. Do not let React component state become the timeline source of truth.
- Trim handles should be edge hit-zones/brackets, not layout-affecting buttons inside clip content. The clip label should not shift when handles appear.
- Clip interaction math must use frames internally and preserve half-open intervals: `[timelineIn, timelineOut)`.
- Trim/drag algorithms must clamp or reject: inverted clips, source-bound exhaustion, composition-bound overflow, and same-track overlaps.
- Snapping is pointer-interaction feedback only unless explicitly adding keyboard snap. Keyboard frame stepping must remain exact and independent.
- Preview math and commit math must share the same pure helper, or tests must prove they cannot diverge.

## Research Gaps / Perplexity Queries

Use these when the implementation direction is unclear or when reworking a broken timeline interaction:

```text
React Electron NLE timeline architecture shared simple editor and advanced multi-track editor same canonical project model, bidirectional sync, non-destructive cuts, trim, split, export EDL, examples and open-source code
```

```text
Professional video editor timeline trim UX edge hit zones local drag preview commit on pointerup React pointer capture stale closures snap collision min duration source bounds implementation patterns
```

```text
How should screen recorder editor simple cut view and advanced NLE timeline share one model camera audio cursor telemetry zoom markers linked clips sync lock export parity
```

```text
Open source React video editor timeline clip trim drag implementation local preview state pure immutable mutations undo redo snapping collision sourceIn timelineIn sourceOut timelineOut
```

```text
Screen recording editor architecture one timeline two toolsets simple recording edit advanced NLE same state cut ranges zoom markers camera PiP cursor telemetry FFmpeg export parity migration
```

```text
React Electron NLE timeline playback wall-clock virtual playhead hidden video elements as frame decoders sourceIn offsets timeline gaps canvas renderer architecture
```

## Important Files

- `packages/timeline-engine/src/zoom-transform.ts`: shared preview/export transform resolution. Keep overlap precedence here so preview and export stay in sync.
- `packages/timeline-engine/src/zoom-transform.test.ts`: regression coverage for active marker precedence.
- `apps/desktop/src/renderer/src/zoom-markers.mjs`: renderer marker helpers for add/remove/range/strength edits.
- `apps/desktop/src/renderer/src/zoom-markers.test.mjs`: marker edit regression coverage.
- `apps/desktop/src/renderer/src/timeline-rail.mjs`: timeline lane placement and zoom layer assignment.
- `apps/desktop/src/renderer/src/timeline-rail.test.mjs`: timeline model and layer assignment coverage.
- `apps/desktop/src/renderer/src/main.tsx`: Timeline panel, stable Inspector controls, and pointer drag/resize wiring.
- `apps/desktop/src/renderer/src/nle/*`: NLE shell, timeline, program monitor, clip mutations, track rendering, keyboard/snap helpers.
- `apps/desktop/src/renderer/src/styled-video-preview.tsx`: shared canvas/program preview. Timeline mode must render from canonical timeline time while hidden videos act only as decoders.
- `packages/frame-resolver/src/resolve-frame.ts`: shared preview-frame resolver. Keep gap/sourceIn/timelineIn behavior canonical here.
- `packages/frame-resolver/src/timeline-frame.ts`: canonical timeline-frame resolver for active clips and empty gap frames.
- `apps/desktop/src/renderer/src/styles.css`: compact panel styling, no-overflow guards, zoom handles/layers.
- `scripts/smoke-ui.mjs` and `apps/desktop/src/main/index.mjs`: UI smoke assertions for handles/no overflow.

## Verification

Run focused checks after zoom timeline changes:

```bash
pnpm --filter @rough-cut/timeline-engine test
node --test apps/desktop/src/renderer/src/zoom-markers.test.mjs apps/desktop/src/renderer/src/timeline-rail.test.mjs
pnpm --filter @rough-cut/project-model build && pnpm --filter @rough-cut/timeline-engine build && pnpm --filter @rough-cut/desktop typecheck
pnpm smoke:ui
```

For export parity changes, also run:

```bash
pnpm smoke:styled-export
```
