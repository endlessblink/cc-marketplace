---
name: dashboard-designer
description: Premium dark-mode admin dashboard designer with RTL Hebrew support, content editor UX patterns, and Tailwind templates. Use when building or redesigning dashboards, admin panels, or control panels.
---

# Dashboard Designer

Premium dark-mode dashboard design system with RTL Hebrew support. Stack-agnostic — works with any framework (Next.js, FastAPI+Jinja2, plain HTML) using Tailwind CSS.

## When to Use

Activate when the user asks to:
- Build or redesign a dashboard, admin panel, or control panel
- Fix dark mode visual hierarchy
- Design content editors, settings pages, or data tables
- Handle RTL Hebrew layouts in admin interfaces

---

## 1. Surface Hierarchy

Use **Tailwind zinc scale** for reliable dark surfaces. Based on shadcn/ui, GitHub Primer, and Material Design research. Avoid `bg-white/[0.02]` or similar low-opacity values — they are below human perception threshold and unreliable with Tailwind CDN.

```
Page background:     bg-zinc-950   (#09090b)
─────────────────────────────────────────────
Sidebar:             bg-zinc-900   (#18181b) + border border-zinc-800
Primary cards:       bg-zinc-900   (#18181b) + border border-zinc-800 rounded-2xl
Active/editor card:  bg-zinc-900   + border border-zinc-700 + shadow-lg shadow-black/25
Sub-panels:          bg-zinc-950   (#09090b) + border border-zinc-700 rounded-xl
Table headers:       bg-zinc-900   (#18181b)
Inputs/textareas:    bg-zinc-950   (#09090b) + border border-zinc-700
─────────────────────────────────────────────
Hover (rows/items):  hover:bg-zinc-800  (#27272a)
Selected:            bg-indigo-500/10 ring-1 ring-indigo-500/25
Focus ring:          focus:ring-2 focus:ring-indigo-500/50
Dropdown menus:      bg-zinc-900 border border-zinc-700
```

### Why solid zinc over opacity
- `bg-white/[0.02]` = 2% white = invisible (Material Design minimum is 5%)
- Tailwind CDN may not reliably render arbitrary bracket opacity values
- Zinc scale is battle-tested (shadcn/ui, GitHub) and guaranteed to render
- Card-to-background contrast is clear: zinc-900 (#18181b) on zinc-950 (#09090b)

---

## 2. Typography Ladder

Clear hierarchy through the zinc color scale. Each level is visually distinct and readable.

```
Page title:     text-xl font-semibold text-zinc-100  (#f4f4f5)
Section head:   text-lg font-semibold text-zinc-100
Body text:      text-sm text-zinc-300               (#d4d4d8)
Secondary:      text-sm text-zinc-400               (#a1a1aa)
Metadata:       text-xs text-zinc-500               (#71717a)
Labels:         text-xs uppercase tracking-wider text-zinc-500 font-medium
Placeholder:    placeholder:text-zinc-600           (#52525b)
─────────────────────────────────────────────
Dividers:       divide-zinc-800  or  border-zinc-800
Section gaps:   space-y-6  (between major sections)
Card padding:   p-5 or p-6
```

---

## 3. Button Hierarchy

Every page should have exactly ONE primary action. All other buttons are secondary or ghost. Dangerous actions always require confirmation.

```
PRIMARY (save/submit):
  bg-zinc-100 text-zinc-900 hover:bg-white font-medium px-5 py-2 rounded-lg cursor-pointer

SECONDARY (AI, filter, export):
  border border-zinc-700 text-zinc-300 hover:bg-zinc-800 px-4 py-2 rounded-lg cursor-pointer

GHOST (cancel, dismiss, nav):
  text-zinc-400 hover:text-zinc-200 hover:bg-zinc-800 px-3 py-2 rounded-lg cursor-pointer

DANGER (delete, reset, regenerate-all):
  text-rose-400 hover:bg-rose-500/10 px-4 py-2 rounded-lg cursor-pointer
  ALWAYS inside overflow menu or with confirmation modal
  NEVER as a peer button next to Save
```

### AI Action Pattern
Replace separate "Add more" + "Regenerate all" buttons with a single dropdown:

```html
<div class="relative" data-dropdown>
  <button onclick="this.nextElementSibling.classList.toggle('hidden')"
          class="border border-white/12 text-white/88 px-4 py-2 rounded-lg
                 hover:bg-white/5 text-sm flex items-center gap-2 cursor-pointer">
    <span class="text-purple-300">✦</span> צור עם AI
    <svg class="w-3.5 h-3.5 opacity-50" fill="none" stroke="currentColor" viewBox="0 0 24 24">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"/>
    </svg>
  </button>
  <div class="hidden absolute top-full mt-1 start-0 w-48 rounded-lg border border-white/10
              bg-[#1a1a2e] shadow-xl shadow-black/40 py-1 z-50">
    <button class="w-full text-start px-4 py-2 text-sm text-white/84 hover:bg-white/[0.05] cursor-pointer"
            onclick="/* generate append 5 */">
      הוסף 5 חדשים
    </button>
    <button class="w-full text-start px-4 py-2 text-sm text-white/84 hover:bg-white/[0.05] cursor-pointer"
            onclick="/* generate append 10 */">
      הוסף 10 חדשים
    </button>
    <div class="my-1 border-t border-white/6"></div>
    <button class="w-full text-start px-4 py-2 text-sm text-rose-300 hover:bg-rose-500/10 cursor-pointer"
            onclick="/* confirm then regenerate all */">
      שכתב הכל מחדש...
    </button>
  </div>
</div>
```

---

## 4. Status Badges

```
Success/Active:  bg-emerald-500/15 text-emerald-300 border border-emerald-500/20
                 text-xs font-medium px-2 py-0.5 rounded-full

Warning:         bg-amber-500/15 text-amber-300 border border-amber-500/20

Error/Danger:    bg-rose-500/15 text-rose-300 border border-rose-500/20

Info:            bg-sky-500/15 text-sky-300 border border-sky-500/20

Neutral:         bg-white/5 text-white/55 border border-white/8
```

### Schedule Badge
For scheduled content, use a compact inline badge:
```html
<span class="text-[11px] uppercase tracking-wide text-white/45
             border border-white/8 rounded-full px-2.5 py-0.5">
  כל יום · 08:00
</span>
```

---

## 5. RTL Hebrew Layout

### Sidebar
- Sidebar on the **RIGHT** side (fixed)
- Main content offset: `me-64` (or `mr-64` for Tailwind < v4)
- Active nav indicator: `border-inline-end` (appears on visual left in RTL)

### Text Direction
- Set `<html dir="rtl" lang="he">` on the document
- Use `text-start` / `text-end` instead of `text-right` / `text-left`
- Use logical properties: `ms-*` `me-*` `ps-*` `pe-*` instead of `ml-` `mr-` `pl-` `pr-`
- Wrap English text, numbers, code in `<bdi>` or `<span dir="ltr">`
- Flex rows flow naturally in RTL — no changes needed

### Mixed Content
```html
<p>המשתמש <bdi>@username</bdi> הצטרף לקבוצה</p>
<p>נשלחו <bdi>42</bdi> הודעות היום</p>
```

---

## 6. Page Templates

### Stats / Overview
```
┌──────────────────────────────────────────────┐
│ Page title + subtitle                        │
├──────┬──────┬──────┬──────┤                  │
│ KPI  │ KPI  │ KPI  │ KPI  │  ← grid-cols-4  │
│ card │ card │ card │ card │                  │
├──────┴──────┼──────┴──────┤                  │
│ Leaderboard │  Upcoming   │  ← grid-cols-2   │
│             │  Events     │                  │
└─────────────┴─────────────┘
```
- KPI cards: large stat number + label + trend indicator (arrow + percentage)
- Each card: `bg-white/[0.02] border border-white/10 rounded-2xl p-5`

### Content Editor (Scheduled Content)
**Anti-patterns to avoid:**
- Tabs per content type
- Info banners / paragraphs explaining each section
- Multiple generate buttons at same level as save

**Correct pattern: Stacked schedule blocks**
```
┌──────────────────────────────────────────────┐
│ Page title: "תוכן מתוזמן"                    │
│                                              │
│ ┌──────────────────────────────────────────┐ │
│ │ הודעות בוקר  [כל יום · 08:00] [ערוץ X]  │ │
│ │──────────────────────────────────────────│ │
│ │ [textarea / item list]                   │ │
│ │──────────────────────────────────────────│ │
│ │ [✦ צור עם AI ▾]              [שמור]     │ │
│ └──────────────────────────────────────────┘ │
│                                              │
│ ┌──────────────────────────────────────────┐ │
│ │ הודעות ערב   [כל יום · 21:00] [ערוץ X]  │ │
│ │──────────────────────────────────────────│ │
│ │ [textarea / item list]                   │ │
│ │──────────────────────────────────────────│ │
│ │ [✦ צור עם AI ▾]              [שמור]     │ │
│ └──────────────────────────────────────────┘ │
│                                              │
│ ┌──────────────────────────────────────────┐ │
│ │ שאלות לדיון  [3x ביום] [ערוצים אקראיים] │ │
│ │ [pill: קטגוריה1] [pill: קטגוריה2] ...    │ │
│ │──────────────────────────────────────────│ │
│ │ [textarea for selected category]         │ │
│ │──────────────────────────────────────────│ │
│ │ [✦ צור עם AI ▾]              [שמור]     │ │
│ └──────────────────────────────────────────┘ │
└──────────────────────────────────────────────┘
```

Each block header:
```html
<div class="flex items-center justify-between">
  <h3 class="text-lg font-semibold text-white/92">הודעות בוקר</h3>
  <div class="flex items-center gap-2">
    <span class="schedule-badge">כל יום · 08:00</span>
    <span class="channel-chip">ערוץ יעדים</span>
  </div>
</div>
```

### Table Page
```html
<!-- Header -->
<div class="flex items-center justify-between mb-6">
  <div>
    <h2 class="text-xl font-semibold text-white/92">חברים</h2>
    <p class="text-sm text-white/55 mt-1">142 חברים רשומים</p>
  </div>
  <button class="primary-btn">+ הוסף</button>
</div>

<!-- Table -->
<div class="rounded-2xl border border-white/10 bg-white/[0.02] overflow-hidden">
  <table class="w-full">
    <thead>
      <tr class="border-b border-white/6">
        <th class="px-5 py-3 text-start text-[11px] uppercase tracking-wide
                   text-white/45 font-medium">שם</th>
        ...
      </tr>
    </thead>
    <tbody class="divide-y divide-white/6">
      <tr class="hover:bg-white/[0.03] transition-colors">
        <td class="px-5 py-3.5 text-sm text-white/84">...</td>
        ...
      </tr>
    </tbody>
  </table>
</div>
```

### Settings Page
- Each settings group in its own card
- Feature toggles: auto-save on change
- Complex forms: explicit save button in card footer

```html
<div class="rounded-2xl border border-white/10 bg-white/[0.02] overflow-hidden">
  <!-- Card header -->
  <div class="px-6 py-4 border-b border-white/6">
    <h3 class="text-base font-semibold text-white/92">הגדרות ספאם</h3>
    <p class="text-xs text-white/45 mt-0.5">כללי זיהוי וחסימה אוטומטית</p>
  </div>
  <!-- Card body -->
  <div class="p-6 space-y-4">
    <!-- form fields -->
  </div>
  <!-- Card footer -->
  <div class="px-6 py-4 border-t border-white/6 flex justify-end">
    <button class="bg-white text-black hover:bg-white/90 font-medium
                   px-5 py-2 rounded-lg text-sm cursor-pointer">שמור</button>
  </div>
</div>
```

### Auto-save Toggle
```html
<label class="flex items-center justify-between p-3 rounded-lg
              hover:bg-white/[0.03] transition-colors cursor-pointer">
  <span class="text-sm text-white/84">הודעות ברוכים הבאים</span>
  <div class="flex items-center gap-2">
    <span class="save-status text-xs text-emerald-300 opacity-0 transition-opacity"></span>
    <input type="checkbox" class="w-4 h-4 accent-sky-500 cursor-pointer"
           onchange="autoSaveToggle(this, 'welcome')">
  </div>
</label>
```

---

## 7. Common Components

### Toast Notifications
```javascript
function showToast(message, type = 'success') {
    const toast = document.createElement('div');
    const bg = type === 'success' ? 'bg-emerald-500/90' : 'bg-rose-500/90';
    toast.className = `fixed top-4 left-4 px-5 py-3 rounded-lg shadow-lg
                       text-white text-sm z-50 backdrop-blur-sm ${bg}
                       animate-[fadeIn_0.2s_ease-out]`;
    toast.textContent = message;
    document.body.appendChild(toast);
    setTimeout(() => {
        toast.style.opacity = '0';
        toast.style.transition = 'opacity 0.3s';
        setTimeout(() => toast.remove(), 300);
    }, 3000);
}
```

### Confirmation Modal
```javascript
function showConfirm(message, onConfirm) {
    const overlay = document.createElement('div');
    overlay.className = 'fixed inset-0 bg-black/60 backdrop-blur-sm z-50 flex items-center justify-center';
    overlay.innerHTML = `
        <div class="bg-[#1a1a2e] border border-white/10 rounded-2xl p-6 max-w-sm mx-4 shadow-2xl">
            <p class="text-sm text-white/84 mb-5">${message}</p>
            <div class="flex gap-3 justify-end">
                <button class="cancel px-4 py-2 text-sm text-white/55 hover:text-white/80
                               hover:bg-white/[0.03] rounded-lg cursor-pointer">ביטול</button>
                <button class="confirm px-4 py-2 text-sm text-rose-300 hover:bg-rose-500/10
                               rounded-lg cursor-pointer">אישור</button>
            </div>
        </div>`;
    overlay.querySelector('.cancel').onclick = () => overlay.remove();
    overlay.querySelector('.confirm').onclick = () => { onConfirm(); overlay.remove(); };
    overlay.onclick = (e) => { if (e.target === overlay) overlay.remove(); };
    document.body.appendChild(overlay);
}
```

### Empty State
```html
<div class="py-12 text-center">
  <p class="text-white/45 text-sm">אין נתונים עדיין</p>
  <button class="mt-3 text-sm text-sky-400 hover:text-sky-300 cursor-pointer">
    הוסף את הראשון
  </button>
</div>
```

---

## 8. Anti-Patterns (Never Do)

| Anti-Pattern | Why | Do Instead |
|---|---|---|
| Tabs for content types | Hides related content, forces mental model switching | Stacked blocks visible together |
| Info banners explaining what sections do | Adds text noise, users skip them | Schedule badges + destination chips inline |
| Multiple generate buttons | Confuses AI actions with save | Single AI dropdown |
| "Regenerate All" as visible button | Destructive action too easy to hit | Inside dropdown with confirmation |
| Different hex per surface layer | Creates patchwork, hard to maintain | Opacity-based system |
| `text-gray-400` / `text-gray-500` in dark mode | Inconsistent tone mapping | `text-white/55` opacity ladder |
| Mixing auto-save and explicit save | Users don't know what's saved | Choose ONE model per page |
| Light-mode classes in dark dashboard | Visual inconsistency, breaks immersion | Always use dark variants |
| Font-mono for Hebrew content | Monospace doesn't work well with Hebrew glyphs | Sans-serif everywhere |
