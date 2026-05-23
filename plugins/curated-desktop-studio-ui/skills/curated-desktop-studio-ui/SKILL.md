---
name: curated-desktop-studio-ui
description: Designs and critiques curated desktop/studio-machine interfaces with OS metaphors, draggable windows, folders, docks, feeds, and playful widgets. Use when working on personal portfolio homepages or product surfaces that should feel like a mature macOS/Raycast/Arc/Obsidian-inspired workspace rather than a generic landing page.
---

# Curated Desktop Studio UI

## Purpose

Use this skill for interfaces that are meant to feel like a personal machine: folders, windows, docks, activity panels, playful utilities, live feeds, and spatial exploration.

The goal is not to turn the page into a normal SaaS homepage. The goal is a curated desktop that feels alive, personal, and usable.

## Core Principle

Treat the page like a staged studio desktop, not a real messy desktop.

Real desktops are cluttered. This aesthetic is selective: a few meaningful objects, clear zones, consistent lighting, and enough empty space for the eye to rest.

## Composition Rules

- Use spatial hierarchy before labels. Position, grouping, contrast, and depth should imply where to start.
- Avoid one giant hero folder unless the product explicitly needs one. Balanced exploration is usually stronger.
- Arrange core folders in a loose arc or stepped grid, not random scatter.
- Place the intended path in the upper/central cluster: Skills or Workflows first, Learning or Start Here nearby.
- Place supporting objects lower or toward the sides: Work, Services, Contact, Archives.
- Keep live feeds and utility widgets as contextual layers, not the main stage.
- Do not add artificial spotlight blobs in empty space. If lighting exists, it must feel like ambient desktop light, not a UI annotation.
- Preserve breathing room around the central object group.

## Object Priority

Use layered priority instead of explicit hierarchy labels.

Primary path objects:
- Slightly stronger material contrast.
- Higher label contrast.
- Subtler but cleaner shadow.
- Better placement, usually upper-center or first in the spatial sequence.

Secondary objects:
- Nearby placement.
- Warm or distinct material, but less contrast than primary.
- Same physical size unless there is a strong product reason to vary size.

Supporting objects:
- Flatter materials.
- Lower saturation.
- Slightly more peripheral placement.

Never write labels like “primary,” “start here,” or “main entry” as visual band-aids unless the product itself would naturally use that language.

## Folder Materials

Folders should feel like premium OS artifacts, not generic 3D icons.

Recommended materials:
- Command / Skills: deep graphite, slate metal, muted blue-violet rim.
- Learning / Start Here: muted brass, aged paper, warm ochre, never bright gold.
- Work / Projects: archival green or desaturated blue-green.
- Weekly / Signal: muted indigo or violet.
- Services / Systems: dark teal or graphite.
- Contact / Personal: violet-gray, graphite, or paper-card material.

Material rules:
- Keep one consistent light direction, usually top-left.
- Use subtle bevels, seams, rim highlights, and grounded shadows.
- Avoid bright Finder blue unless intentionally referencing stock macOS.
- Avoid glossy toy-like gradients.
- Keep glyphs readable: high-contrast icon color plus a small shadow on lighter folders.
- Do not hide the folder silhouette behind the icon or decoration.

## App And Shortcut Icons

Desktop shortcut icons must be treated as a real icon family, not as buttons with generic line glyphs.

Rules:
- Never hand-code quick pictograms as the final icon set. Inline CSS/SVG primitives are acceptable only for wireframes and must be replaced before presenting as design quality.
- Do not repeat the same glyph across multiple shortcuts in a workspace. Each shortcut needs semantic specificity: dictionary, guide, video reel, code terminal, dashboard, automation graph, profile, contact, etc.
- Do not use third-party product/app icons for site sections unless the section literally opens that product. Borrowing Obsidian/Claude/DaVinci/etc. icons for generic navigation creates brand mismatch.
- If the project already has high-quality icon assets, prefer them only when the icon semantically matches the destination. Do not reuse a nearby-looking asset just because the material style matches.
- If a coherent icon family does not exist, stop and generate or source a full matching set before implementing. Use a single prompt/style sheet for the whole set: same camera, material, lighting, corner radius, shadow depth, and background transparency.
- Good OS-style icons are object-like: large silhouette, depth, bevels, inner highlights, material texture, and one clear metaphor. They are not tiny line icons inside identical rounded squares.
- App/shortcut icons should render at 72-96px with enough detail to feel premium at desktop size and still read when scaled down on mobile.

Quality gate before shipping icons:
- Put all icons in a row at equal size. If any look like a different pack, reject the set.
- Compare against the existing dock/folder asset quality. If the new icons look flatter, cheaper, or more placeholder-like, reject them.
- Verify semantic match one by one. If a user cannot guess the destination category from the icon and label together, redesign it.

## Windows And Widgets

Windows carry information density, so they easily dominate.

Rules:
- About windows should be quieter than the folder workspace unless the page is explicitly an About page.
- Activity feeds should feel like a live side panel: useful, but not louder than the core folder cluster.
- Games and playful widgets are allowed. They are personality objects, not disposable noise. Keep them visible but peripheral.
- Do not resize interactive widgets in ways that can break their internal logic.
- If draggable offsets persist, avoid changing default widget position/size without resetting or versioning drag state.

## Dock Rules

- The dock supports navigation; it should not carry the whole hierarchy.
- Order dock icons according to the intended journey.
- Use hover magnification and labels for affordance, not constant emphasis.
- Avoid glowing one dock item permanently unless it represents an active app.

## Motion

- Use motion to make objects feel physical: settle, lift, press, open, minimize.
- Keep entry animations staggered in path order: primary path first, secondary next, support after.
- Avoid bouncy toy motion. Prefer confident ease-out curves.
- Do not animate layout properties. Use transforms and opacity.
- Respect reduced motion.

## Copy And Typography

- Use copy that explains the machine without turning it into a marketing landing page.
- Prefer concise, direct labels: Skills, Learning, Work, Weekly, Services, Contact.
- Hebrew text should be readable and right-aligned where natural.
- English app terms, commands, and product names can remain English.
- Avoid generic AI copy such as “unlock potential,” “future of work,” “revolution,” or “AI platform.”

## Anti-Patterns

Reject these immediately:
- Replacing the desktop with a generic landing page.
- Equal visual weight everywhere.
- Random folder scatter with no spatial sequence.
- Bright gold, neon purple, or generic AI gradient materials.
- Spotlight blobs used as hierarchy hints.
- Labels that explain hierarchy instead of designing hierarchy.
- Oversized About panels that compete with navigation.
- Hiding personality widgets because they are inconvenient.
- Generic SaaS card grids inside windows.
- Glassmorphism used as decoration rather than as OS material.
- Generic repeated rounded-square icons with swapped line glyphs.
- Hand-coded placeholder SVG icons presented as final app icons.
- Reusing third-party app logos as navigation icons for unrelated site sections.
- Reusing existing assets that match material style but do not match the destination meaning.

## Review Checklist

Before shipping, answer:

- Does the page still feel like a personal machine, not a generic homepage?
- Can a visitor infer the first two paths within five seconds?
- Are folders grouped intentionally rather than scattered?
- Are windows and widgets supporting the desktop instead of stealing it?
- Are all folder glyphs readable?
- Do materials share one lighting system?
- Did any visual trick feel like an annotation rather than an object in the world?
- Does the mobile version preserve the metaphor without squeezing a desktop onto a phone?

## Current Reference Direction

Useful reference families:

- macOS Finder: familiar windows, dock, folders, and spatial affordances.
- Raycast: command-first clarity, dark surfaces, crisp hierarchy.
- Arc: spaces, calm chrome, restrained personal browsing metaphor.
- Obsidian: graph/folder/pane knowledge workspace, personal machine feeling.
- Linear: density control, clean panels, confident typography.
- Panic apps: tasteful skeuomorphic objects and mature icon materials.

Use these as principles, not as skins.
