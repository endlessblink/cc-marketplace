---
name: obsidian-memory-reliability
description: Designs and implements robust durable memory for Obsidian AI plugins. Use when working on Ghost in the Vault memory capture, AI Memory/Core.md, protected memory namespaces, bilingual Hebrew/English triggers, memory verification, migration of unsupported memory notes, or preventing LLM hallucinated memory saves.
---

# Obsidian Memory Reliability

## Core Architecture

Use a strict plugin-managed memory namespace.

- `AI Memory/` is owned by the plugin `MemoryManager`.
- LLM file actions must not create, append, replace, delete, rename, or move files under `AI Memory/`.
- Read/open operations are allowed for inspection, but persistence goes only through `MemoryManager.save(...)` or equivalent.
- Start with `AI Memory/Core.md` only for reliable v1 storage.
- Use `domain:` metadata for organization instead of splitting files early.
- Add optional domain files later only with an explicit index and loader guarantees.

## Hard Invariants

- Only `MemoryManager` writes durable memory.
- Every save performs read, append, reread, and verify before reporting success.
- No successful save message appears unless verification passes.
- Parser trusts only canonical structured memory lines in designated files.
- Unsupported prose notes under `AI Memory/` are ignored by loading and flagged in UI/audit.
- User memory triggers bypass the LLM and go directly to plugin capture.
- Hebrew, English, and mixed Hebrew/English are first-class inputs for all memory triggers and tests.

## Canonical Memory Line Format

Use one structured Markdown bullet per memory:

```markdown
- domain: work; createdAt: 2026-05-19T20:38:00Z; source: chat:user-directive; verifiedAt: 2026-05-19T20:38:05Z; originalLang: mixed; tags: critical,task-routing; Task Storage Rule: Always use X for Y. Context: ...
```

Required fields:

- `domain`: `global`, `personal`, `work`, or `project:<slug>`.
- `createdAt`: ISO timestamp.
- `source`: `chat:user-directive`, `chat:exchange`, `migrated:<path>`, or similar.
- `verifiedAt`: ISO timestamp set only after reread verification.
- text: single-line free text after metadata.

Recommended fields:

- `originalLang`: `en`, `he`, or `mixed`.
- `tags`: comma-separated tags such as `critical`, `task-routing`, `workflow`, `bilingual`.

Rules:

- Preserve Hebrew, emoji, and paths.
- Normalize multiline input into one line with ` | ` unless a future block format is explicitly implemented.
- Keep parsing strict enough to avoid arbitrary prose becoming memory.

## Protected Namespace Enforcement

Action runner policy:

- Block `create`, `append`, `replace`, `delete`, `rename`, and move-like actions whose target path starts with `AI Memory/`.
- For rename/move, check both `from` and `to` paths.
- Return a clear message: `AI Memory is plugin-managed. Use memory capture instead.`
- Add regression tests proving `ghost-action` cannot write `AI Memory/Task Storage Rule.md`.

## Memory Save UX

Use deterministic plugin-generated replies, localized to the user's input language.

Short clear memory:

```text
Remembered and verified in AI Memory/Core.md.
domain: work; Task Storage Rule: ...
```

Hebrew equivalent:

```text
זכרתי ואימתתי ב-AI Memory/Core.md.
domain: work; ...
```

Do not let the LLM generate `Memory Updated`, `Remembered`, or similar durable-save claims.

Preview/confirmation is required when:

- memory text exceeds 800 characters,
- input contains newlines,
- trigger is vague, such as `remember this` or `תזכור את זה`.

For short, explicit, single-line memory requests, save immediately and verify.

## Remember This Referents

Default for bare `remember this` after a chat exchange:

- Save both previous user message and assistant answer as `source: chat:exchange`.
- Format text as `User: ... | Assistant: ...`.
- If the exchange is long or ambiguous, show a preview/choice instead of saving silently.
- Do not save assistant hallucinated memory cards as the referent.

## Bilingual Trigger Strategy

Memory triggers must include English, Hebrew, and mixed phrasing.

Examples:

- `remember ...`
- `save this ...`
- `remember את זה`
- `תזכור ...`
- `תזכרי ...`
- `שמור ...`
- `שמרי ...`
- `תשמור ...`
- `תשמרי ...`

Domain inference must include English and Hebrew terms:

- work: `work`, `project`, `task`, `client`, `עבודה`, `פרויקט`, `פרוייקט`, `משימה`, `משימות`, `לקוח`.
- personal: `personal`, `home`, `private`, `family`, `אישי`, `אישית`, `פרטי`, `בית`, `משפחה`.

Add pure Hebrew, pure English, mixed Hebrew/English, RTL, paths, and emoji fixtures for every natural-language parser change.

## Unsupported Prose Memory Notes

Do not parse arbitrary prose notes as memory.

On load/audit, scan `AI Memory/*.md` for files that contain no canonical memory lines.

- Show them as `Not loaded as memory` in Memory Review.
- Offer one-click `Migrate to Core.md`.
- Migration extracts the useful content into canonical line(s), writes with `source: migrated:<path>`, verifies, then marks the original with frontmatter/comment such as `migrated-to-core: true`.
- Do not auto-delete migrated notes.
- Do not auto-migrate without user action.

## Always-Loaded Critical Memory

Memories tagged `critical` or `task-routing` load regardless of active domain.

Use this for workflow routing rules such as task storage paths.

## Test Plan

Required tests for reliability work:

- Parser accepts canonical lines with metadata and rejects malformed prose.
- Hebrew and mixed memory triggers bypass the LLM and call `MemoryManager`.
- `remember this` uses the intended previous exchange or asks for confirmation.
- Memory save writes, rereads, verifies, and only then reports success.
- Verification failure reports an error and does not show success.
- `ghost-action` writes under `AI Memory/` are blocked.
- Prose-only notes under `AI Memory/` are surfaced as ignored/unloaded.
- Migration writes canonical Core entries and marks originals without deleting them.

## Implementation Priority

1. Protected `AI Memory/` namespace.
2. `Core.md`-only verified saves.
3. Deterministic localized save replies.
4. Unsupported prose-file warnings.
5. One-click migration.
6. Critical/task-routing always-loaded tags.
7. Optional domain files with explicit index.
