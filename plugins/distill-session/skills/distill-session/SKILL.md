---
name: distill-session
description: Distill the current conversation into a reusable Markdown skill at `~/.codex/skills/sessions/<topic>.md` so future sessions can replay the approach instead of re-deriving it. Use when a substantial multi-step task wraps up (major refactor, architecture change, debugging investigation, multi-phase build) or when Noam says "save this session", "make a playbook from this", "distill what we did", "save the approach", "remember how we did this", or invokes `/distill-session [<topic-slug>]`. Cross-tool (Claude Code, Codex, OpenCode).
metadata:
  short-description: Auto-generate a session-summary skill so the approach is reusable in future sessions
  owner: Noam
  created: 2026-05-16
  inspired_by: Nous Research Hermes Agent auto-generated skill files (hermes-agent.nousresearch.com)
---

# /distill-session — Persist the approach of this conversation

## What this does

Extracts the **decisions, rationale, commands, and edge cases** of the current conversation into a Markdown skill file under `~/.codex/skills/sessions/`. Future sessions (in any tool — Claude Code, Codex, OpenCode) can load the skill and replay the approach without re-deriving it from scratch.

**This is not a transcript.** It's a structured playbook: what was the goal, what decisions were made, why, what commands work, what gotchas to watch.

## When to use

Trigger when:
- A substantial multi-step task wraps up (architecture change, multi-phase build, complex debugging).
- The conversation produced a non-obvious pattern that future-you would benefit from inheriting.
- Noam says "save this session" / "make a playbook" / "distill what we did" / "remember how we did this" / "save the approach".
- `/distill-session [<topic-slug>]` is invoked.

**Skip** for:
- Trivial single-edit tasks ("fix typo", "rename variable").
- Discussion-only sessions with no concrete artifacts.
- Sessions where Noam already invoked `/teach-bot` for the durable learnings (those went to `operator_prefs.md` and don't need a session file).

## How to apply

1. **Identify the topic slug.** Short kebab-case name reflecting the session's central deliverable (e.g., `unify-operator-preferences`, `fix-trivia-rsvp-gate`, `migrate-vps-to-doppler`). If Noam supplied one, use it. Otherwise propose one and confirm.

2. **Extract structure from the conversation.** The skill file has a fixed shape (template below). Pull from chat:
   - **Goal**: one sentence — what was the user trying to achieve?
   - **Decisions**: bulleted list of non-obvious choices made, each with a **Why** line. Include rejected alternatives if the rationale was important.
   - **Files touched**: paths with one-line summaries.
   - **Commands that worked**: copy-pasteable shell incantations (deploy script, test runs, SSH, curl).
   - **Edge cases / gotchas**: things that bit us during the session and how we routed around them.
   - **Verification**: how we confirmed the work was done (logs, endpoints, tests).
   - **What we'd do differently**: honest retrospective if applicable.

3. **Show Noam the draft** before writing. Two or three sentences plus the structure outline. Get explicit approval.

4. **Write the file** at `~/.codex/skills/sessions/<topic-slug>.md` with frontmatter (template below). Symlink to `~/.claude/skills/sessions/<topic-slug>.md` so Claude Code discovers it.

5. **Confirm to Noam**: file path written, line count, what trigger phrases will load it in future sessions.

## File template

```markdown
---
name: session-<topic-slug>
description: <One-sentence summary of the session's deliverable, with trigger phrases for when this should load. Be specific about the problem domain so the skill auto-loads correctly.>
metadata:
  short-description: <≤80 char summary>
  owner: Noam
  session_date: <YYYY-MM-DD>
  context: <project name, e.g. my-app, my-bot>
  origin: distilled via /distill-session
---

# Session: <Topic Title>

## Goal
<One sentence — what Noam was trying to achieve.>

## Decisions (and why)
- <Decision>. **Why:** <reason>. **Rejected:** <alt + why>.
- <Decision>. **Why:** <reason>.

## Files touched
- `path/to/file.ext` — <one line>
- ...

## Commands that worked
```bash
<command 1>
<command 2>
```

## Edge cases / gotchas
- <gotcha>: <how we worked around it>
- ...

## Verification
- <how we confirmed it worked>

## Open follow-ups (if any)
- <thing we knew we'd want to revisit>

## Sources
- <links to research / docs / plan files that informed this>
```

## What NOT to do

- Don't write a chat transcript. Distill, don't paraphrase the whole conversation.
- Don't invent decisions that weren't actually made. If the session was exploratory, say so.
- Don't include credentials, tokens, or personal info from logs.
- Don't write the file silently — always confirm the draft with Noam.
- Don't auto-promote a session skill into `config/operator_prefs.md`. Those are different scopes: `operator_prefs.md` = durable preferences; session skills = reusable playbooks.

## Why this exists

Inspired by Nous Research's Hermes Agent, which auto-generates skill files from complex sessions so the approach compounds across time. Without distillation, every multi-hour debugging investigation or architecture build dissolves the moment the chat ends — the model in the next session re-derives everything from CLAUDE.md and the codebase alone. With it, future-you inherits a playbook that says "this is how we already solved it." Operator-approved write keeps it explicit (matches the `/teach-bot` discipline).
