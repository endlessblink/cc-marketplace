---
name: rtl-design
description: RTL (right-to-left) design patterns for Hebrew/Arabic apps with mixed LTR content. Use when building or fixing RTL layouts, forms, inputs, or mixed-direction text. Triggers on RTL issues, Hebrew layout work, bidirectional text problems.
---

# RTL Design Skill

Correct RTL implementation for Hebrew/Arabic applications with mixed LTR content (URLs, emails, brand names, code).

## Core Principle

**RTL is about document flow direction, NOT about pushing everything to the right.** In a proper RTL layout:

- Text and elements flow **from right to left**
- Labels are **start-aligned** (which resolves to right in RTL)
- Inputs stretch full width — their **internal text direction** depends on content type
- URLs, emails, phone numbers are **always LTR** inside their inputs
- Hebrew text fields use `dir="auto"` to auto-detect direction from first character

## The Decision Matrix

### Input `dir` Attribute by Field Type

| Field Type | Content Language | `dir` Value | Input `text-align` |
|---|---|---|---|
| Name, title, heading | Hebrew | `dir="auto"` | Inherited (start) |
| Description, bio, textarea | Mixed/Hebrew | `dir="auto"` | Inherited (start) |
| URL (any) | Always English | `dir="ltr"` | `text-left` |
| Email | Always English | `dir="ltr"` | `text-left` |
| Phone number | Numerals/LTR | `dir="ltr"` | `text-left` |
| Social media handle | Always English | `dir="ltr"` | `text-left` |
| Code/slug/identifier | Always English | `dir="ltr"` | `text-left` |
| Price/number | Numerals | `dir="ltr"` | `text-left` |
| Brand name (English) | English | `dir="ltr"` | `text-left` |
| Freeform (unknown) | Unknown | `dir="auto"` | Inherited (start) |

### Labels and Descriptions

**Labels NEVER need a `dir` attribute.** They inherit from `<html dir="rtl">`.

```html
<!-- CORRECT: label inherits RTL from document -->
<label class="text-sm font-medium">שם הקורס</label>

<!-- WRONG: don't hardcode direction on labels -->
<label class="text-sm font-medium text-right" dir="rtl">שם הקורס</label>
```

**Description text** (helper text below inputs) also inherits RTL. No `dir` needed.

### English Labels in RTL Context

Labels that are English brand names (Instagram, Facebook, YouTube) in an RTL form should still be start-aligned (right). The label IS the field identifier — its position follows document flow, not its language.

```html
<!-- Correct: English label, start-aligned in RTL document -->
<label class="text-sm font-medium">Instagram</label>
<input type="url" dir="ltr" class="text-left" placeholder="https://instagram.com/page" />
```

## HTML Patterns

### Hebrew Text Field

```html
<div class="grid gap-2">
  <label class="text-sm font-medium">שם הקורס</label>
  <input type="text" dir="auto" placeholder="הכנסו שם קורס" />
  <p class="text-sm text-muted-foreground">שם הקורס יוצג בדף הבית</p>
</div>
```

### URL / Email / Phone Field

```html
<div class="grid gap-2">
  <label class="text-sm font-medium">כתובת אתר</label>
  <input
    type="url"
    dir="ltr"
    class="text-left placeholder:text-left"
    placeholder="https://example.com"
  />
  <p class="text-sm text-muted-foreground">כתובת האתר של הקורס</p>
</div>
```

### Mixed Textarea

```html
<div class="grid gap-2">
  <label class="text-sm font-medium">תיאור</label>
  <textarea dir="auto" placeholder="תיאור הקורס..."></textarea>
</div>
```

## Tailwind CSS Rules

### Always Use Logical Properties

| Use This (Logical) | NOT This (Physical) | Why |
|---|---|---|
| `text-start` | `text-right` | Auto-flips with direction |
| `text-end` | `text-left` | Auto-flips with direction |
| `ms-2` | `mr-2` | Margin-start = margin-right in RTL |
| `me-2` | `ml-2` | Margin-end = margin-left in RTL |
| `ps-4` | `pr-4` | Padding-start |
| `pe-4` | `pl-4` | Padding-end |
| `start-0` | `right-0` | Position-start |
| `end-0` | `left-0` | Position-end |
| `rounded-s-md` | `rounded-r-md` | Border-radius-start |
| `border-s` | `border-r` | Border-start |

**Exception:** When you INTENTIONALLY override direction (e.g., `dir="ltr"` input), use physical properties `text-left` since you're explicitly in LTR context.

### Flex and Grid

Flexbox and Grid **automatically reverse** in RTL. A `flex-row` will flow right-to-left. Do NOT manually reverse with `flex-row-reverse` in RTL — that would make it LTR again.

```html
<!-- In RTL: icon appears on right, text on left — correct -->
<div class="flex items-center gap-2">
  <Icon />
  <span>הגדרות</span>
</div>
```

## Radix UI / shadcn Integration

### The Radix Direction Problem

Radix UI components auto-detect direction but can fail in SSR/hydration. **Always pass `dir` explicitly to Radix primitives that render wrapper elements:**

- `<Tabs dir="rtl">` — Radix Tabs sets `dir` on its root element
- `<DialogContent dir="rtl">` — Dialog portals render outside the DOM tree
- `<AlertDialogContent dir="rtl">` — Same portal issue
- `<SelectContent dir="rtl">` — Select dropdown portals
- `<DropdownMenuContent dir="rtl">` — Dropdown portals

**Fix at component level** (preferred): Default `dir="rtl"` in your shadcn wrappers:

```tsx
// In src/components/ui/tabs.tsx
function Tabs({
  dir = "rtl", // Default for Hebrew app
  ...props
}: React.ComponentProps<typeof TabsPrimitive.Root>) {
  return <TabsPrimitive.Root dir={dir} {...props} />
}
```

### Portaled Components

Dialog, AlertDialog, Select, DropdownMenu, Popover — these render via React portals **outside** your `<div dir="rtl">` wrapper. They inherit from `<html dir="rtl">` but Radix may override. Always set `dir="rtl"` on their Content component.

## Mixed-Direction Display Text

For read-only text that mixes Hebrew and English (not inputs):

```html
<!-- Use <bdi> for unknown-direction inline text -->
<p>הפרויקט נמצא ב-<bdi>github.com/user/repo</bdi></p>

<!-- Use <span dir="ltr"> for known-LTR inline text -->
<p>צפו בסרטון ב-<span dir="ltr">YouTube</span></p>
```

**Never use `unicode-bidi` on form inputs.** Use the `dir` HTML attribute instead.

## Common Mistakes

### 1. Setting `dir="rtl"` on everything
Only `<html>` needs `dir="rtl"`. Elements inherit it. Override only where content is LTR.

### 2. Using `text-right` instead of `text-start`
`text-right` is physical. `text-start` is logical and auto-flips. Use `text-start`.

### 3. Missing `dir="ltr"` on URL/email inputs
URLs typed right-to-left are broken. Always `dir="ltr"` + `text-left` on LTR-content inputs.

### 4. Forgetting `dir` on Radix portaled components
Dialogs, selects, dropdowns render outside your RTL wrapper. Always pass `dir="rtl"`.

### 5. Using `mr-` / `ml-` instead of `ms-` / `me-`
Physical margins don't flip. Logical margins do. Use `ms-` and `me-`.

### 6. Adding `dir="auto"` to inputs that are ALWAYS one direction
If a field always contains URLs, use `dir="ltr"` — not `dir="auto"`. Auto-detection can misfire.

## Checklist Before Shipping

- [ ] `<html dir="rtl" lang="he">` is set
- [ ] Labels use `text-start` (or inherit), never `text-right`
- [ ] Hebrew text inputs have `dir="auto"`
- [ ] URL/email/phone/code inputs have `dir="ltr"` + `text-left`
- [ ] Radix portaled components (Dialog, Select, etc.) have `dir="rtl"` on Content
- [ ] Radix Tabs has `dir="rtl"` passed to Root
- [ ] Tailwind uses logical properties (`ms-`, `me-`, `ps-`, `pe-`, `text-start`)
- [ ] Flex/grid layouts are NOT manually reversed (they auto-reverse in RTL)
- [ ] Placeholders in LTR inputs have `placeholder:text-left`
