---
name: vue-sortable-grouped-dnd
description: Implement grouped drag-and-drop in Vue 3 with SortableJS, vuedraggable, auto-scroll, cross-group transfer, and related edge-case guidance.
---

# Vue SortableJS Grouped Drag-and-Drop Skill

Use when implementing drag-and-drop between grouped lists in Vue 3 with vuedraggable or vue-draggable-plus.

## Key Principles

1. **Use SortableJS (via vuedraggable) instead of native HTML5 DnD** for grouped lists. Native DnD lacks animation, auto-scroll, and smooth cross-group transfer.

2. **ALWAYS use `:force-fallback="true"`** (explicit boolean binding, never bare attribute — see BUG-1335). This activates SortableJS's own drag emulation which enables:
   - Working auto-scroll on desktop Chrome/Firefox
   - `chosenClass` and `dragClass` applying correctly
   - `animation` property working smoothly

3. **Cross-group transfer** uses the `group` option:
   ```vue
   <draggable
     v-model="group.tasks"
     :group="{ name: 'tasks', pull: true, put: true }"
     item-key="id"
   >
   ```
   All `<draggable>` instances with the same group name can transfer items between each other.

## Auto-Scroll Configuration

```vue
<draggable
  :scroll="scrollContainerRef"
  :scroll-sensitivity="60"
  :scroll-speed="14"
  :force-fallback="true"
  :force-auto-scroll-fallback="true"
  :bubble-scroll="true"
>
```

- `scroll`: ref to the scrollable container element (NOT window)
- `scroll-sensitivity`: px from edge to start scrolling (default 30 is too small, use 60)
- `scroll-speed`: px per frame (default 10 is sluggish, use 12-16)
- `force-auto-scroll-fallback`: MUST be true for Chrome/Firefox
- `bubble-scroll`: also scroll parent containers

## Visual Feedback (CSS must be GLOBAL, not scoped)

```css
/* Drop indicator line — collapse ghost to thin line */
.task-ghost {
  height: 2px !important;
  background: var(--brand-primary);
  border-radius: 1px;
  opacity: 1 !important;
  overflow: hidden;
  padding: 0 !important;
  margin: 2px 0;
}

/* Source item dimmed while dragging */
.task-chosen {
  opacity: 0.4;
}

/* Floating drag image */
.task-dragging {
  opacity: 0.9;
  transform: rotate(1.5deg) scale(1.02);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.4);
}
```

SortableJS adds these classes directly to DOM — scoped CSS won't match them.

## Events for Persistence

```vue
<draggable
  @add="onTaskAdded($event, group)"
  @remove="onTaskRemoved($event, group)"
  @update="onTaskReordered($event, group)"
>
```

- `@add` fires on the RECEIVING list — use this to persist group change
- `@remove` fires on the SOURCE list
- `@update` fires on reorder within same list
- v-model handles the array mutation; you just persist the side effect

```ts
function onTaskAdded(evt: any, targetGroup: Group) {
  const task = targetGroup.tasks[evt.newIndex]
  updateTaskGroup(task.id, targetGroup.id)
}
```

## Empty Groups

ALWAYS render an empty drop zone when `group.tasks.length === 0`:
```vue
<div v-if="group.tasks.length === 0" class="empty-drop-zone" style="min-height: 40px;">
  Drop tasks here
</div>
```
Without this, SortableJS has no target area for empty groups.

## Common Pitfalls

- **Bare boolean attrs on vuedraggable** pass empty string via `$attrs` (BUG-1335). Always use `:attr="true"`.
- **Don't mix** SortableJS `force-fallback` with raw HTML5 `@dragenter`/`@dragover` on same elements.
- **Ghost class won't work with scoped CSS** — use global styles or `:deep()`.
- **`vuedraggable@4.1.0`** (Vue 3 port) is unmaintained since 2022 but functional. `vue-draggable-plus` is the actively maintained alternative with same API.

## References

- [vue-draggable-plus](https://github.com/Alfred-Skyblue/vue-draggable-plus) — active fork
- [SortableJS AutoScroll plugin](https://github.com/SortableJS/Sortable/blob/master/plugins/AutoScroll/README.md)
- [SortableJS options](https://github.com/SortableJS/Sortable#options)
