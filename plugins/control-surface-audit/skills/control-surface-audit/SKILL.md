---
name: control-surface-audit
description: Audit admin dashboards, internal tools, schedulers, workflow builders, bot dashboards, and control panels for hidden reliability holes, misleading UI states, missing lifecycle guarantees, unobserved background jobs, and absent vertical tests. Use when a fast-built project feels hollow, fragile, or untrustworthy and the user wants to discover unknown broken flows before stabilizing or rewriting.
---

# Control Surface Audit

## Overview

Audit a control surface as an operational system, not just a UI or codebase. A control surface is any dashboard, admin panel, internal tool, scheduler UI, workflow builder, bot console, AI action panel, or CRUD interface where user actions are expected to produce durable backend behavior or external side effects.

This skill is read-only by default. Do not edit files, mutate data, run production actions, send messages, trigger jobs, claim tasks, or change statuses unless the user explicitly asks for implementation after the audit.

The goal is to answer: what promises does this control surface make, where can those promises silently fail, and what tests or observability would prove the system is trustworthy?

## When To Use

Use this skill when the user says or implies any of the following:

- A dashboard, admin panel, scheduler, planner, workflow builder, or internal tool feels hollow or fragile.
- A UI card, button, or state looked real but did not produce the expected backend behavior.
- A project was built quickly and may have many hidden holes.
- Background jobs, scheduled actions, AI-generated actions, queues, webhooks, bot sends, or external integrations might not be observable or tested.
- The user is considering a rewrite because fixing one thing seems to break another.
- The user wants to find unknown unknowns across a complex dashboard.

Do not use this skill for ordinary UI polish, standalone code review, or narrow bug fixes unless the bug points to broader lifecycle or trust failures.

## Core Concepts

### Control Surface

Any interface where a human operates a system: creates rows, schedules work, sends messages, approves AI suggestions, changes configuration, starts jobs, runs automations, or triggers external effects.

### Trust Gap

A mismatch between what the UI implies and what the system can prove. Examples:

- Preview content looks scheduled.
- A success toast appears before durable state exists.
- A scheduled row exists but the worker cannot see it.
- A background job fails only in logs.
- A button silently does nothing on a missing dependency.
- A dashboard hides failed/skipped rows.
- Two code paths create similar records with different required fields.
- AI suggestions look final but are drafts or uncommitted payloads.

### Lifecycle Invariant

A rule that must always be true for the product to be trustworthy. Examples:

- If the UI says `scheduled`, a persisted scheduled record must exist.
- Every due background action must end as `succeeded`, `failed`, `skipped`, or remain pending with a visible reason.
- Every external side effect must have a durable result state.
- Preview, draft, scheduled, running, succeeded, failed, skipped, and cancelled must be visually and technically distinguishable.

## Inputs To Inspect

Prefer project-specific evidence over generic heuristics.

- Project rules: `AGENTS.md`, `CLAUDE.md`, `README.md`, architecture docs, release notes, runbooks.
- Dashboard/control UI: routes, templates, pages, components, forms, buttons, modals, drawers, client-side state, action handlers.
- APIs: REST endpoints, server actions, RPC handlers, websocket handlers, webhook receivers.
- Persistence: DB schemas, migrations, repositories, models, ORM entities, status fields, audit/event tables.
- Background execution: cron jobs, schedulers, queues, workers, job runners, pollers, task processors, bot loops.
- External side effects: Telegram/WhatsApp/email sends, payments, deployments, file writes, webhooks, AI calls, third-party APIs.
- Observability: logs, diagnostics endpoints, admin panels, health checks, audit trails, dead-letter queues, metrics, alerts.
- Tests: e2e tests, integration tests, unit tests, fixtures, mocks, contract tests, scheduler tests, webhook tests.
- Runtime state when safe: local DB rows, local logs, local diagnostics endpoints, non-mutating API reads.
- Worktree state: git status and recent diffs only as context; do not revert or overwrite unrelated changes.

For this user's projects, if Watchpost is available or project instructions mention it, check Watchpost before manually parsing task files. Always pass the current repo as `cwd` for repo-scoped Watchpost APIs.

## Discovery Workflow

### 1. Define Scope

State the audited control surface and any exclusions in one sentence. If the user did not specify a scope, audit the most safety-critical dashboard/control flows first.

### 2. Discover Entry Points

Find control-surface pages and actions:

- UI routes, page files, templates, components, modals, drawers, forms, buttons.
- API endpoints or server actions called by those controls.
- Background jobs or workers that consume records created by the UI.
- External integrations those jobs call.

Prefer targeted file search and content search. Avoid reading the whole repository blindly.

### 3. Build Action Inventory

For each meaningful user action, map the full vertical path:

`User action -> client state -> API/server action -> validation -> DB/state mutation -> background owner -> external side effect -> result state -> UI feedback`

Record unknowns explicitly instead of filling gaps with assumptions.

### 4. Build Lifecycle State Map

List every state the flow can enter, including implicit states. Normalize vocabulary when possible:

- `preview`
- `draft`
- `queued`
- `scheduled`
- `running`
- `succeeded` or domain-specific success such as `sent`
- `failed`
- `skipped`
- `cancelled`
- `unknown`

Flag overloaded states, missing terminal states, and states that exist in UI but not persistence or vice versa.

### 5. Identify Trust Gaps

For each action and state, ask:

- What does the UI imply happened?
- What durable evidence proves it happened?
- What background actor owns the next step?
- How does the user know the next step succeeded or failed?
- What happens if the worker is down, credentials are missing, routing is wrong, validation rejects late, or the external API fails?
- Does any path show success before the durable side effect exists?
- Can preview/draft/generated content be mistaken for committed/scheduled content?
- Can the same thing be created through multiple paths with inconsistent required fields?

### 6. Define Invariants

Write project-specific rules that must be enforced by tests or runtime checks. Good invariants are specific, falsifiable, and tied to user trust.

Examples:

- Calendar event with `status=scheduled` must correspond to one persisted scheduled row.
- A due scheduled row must be selected by exactly one scheduler query.
- A send attempt must update durable status to `sent`, `failed`, or `skipped`.
- A failed background action must be visible in the dashboard without reading logs.
- A generated AI suggestion must be clearly `preview` or `draft` until committed.
- Editing a record must not silently change its executable type unless an explicit coercion rule is visible and tested.

### 7. Review Tests Against Promises

Classify tests by promise coverage, not by file count.

- Vertical tests: user action -> persisted state -> worker sees -> external side effect mocked -> terminal state visible.
- Contract tests: API request/response and validation behavior.
- State-machine tests: allowed transitions and impossible transitions.
- Regression tests: previous trust gaps cannot recur.
- Smoke/e2e tests: dashboard interaction works in a browser-like environment when relevant.

Passing tests are not enough. Identify which promises have no test that would fail if the promise broke.

### 8. Recommend Stabilize vs Rewrite

Do not recommend a rewrite as an emotional response to complexity. Recommend a rewrite only when evidence shows one or more of these are true:

- State ownership is fundamentally ambiguous across UI/API/worker/persistence.
- The critical flow cannot be tested without excessive mocking or production side effects.
- Multiple parallel implementations create the same domain object with incompatible semantics.
- The cost to stabilize the current slice exceeds replacing a clearly bounded module.
- The current architecture prevents basic observability of success/failure.

Prefer a bounded stabilization or module replacement over a whole-project rewrite.

## Risk Scoring

Score each gap from 1 to 5. Use the score to make tradeoffs explicit; do not pretend it is objective.

- User impact: how badly user trust or product value is damaged.
- Silent failure risk: whether failure can happen without visible evidence.
- State ambiguity: how unclear ownership or lifecycle is.
- Side-effect risk: whether external systems, users, money, data, or production actions are affected.
- Frequency: how often the flow is used or generated automatically.
- Test gap: whether a regression would currently be caught.
- Fixability: whether the fix is small, bounded, and safe.

Prioritize high user impact, high silent failure risk, high side-effect risk, and missing tests.

## Output Format

Use this structure unless the user asks for something lighter.

### Executive Verdict

State whether the control surface is mostly trustworthy, partially trustworthy with known gaps, or currently untrustworthy for critical flows. Include confidence and the single biggest reason.

### System Map

Briefly map the main runtime path(s): UI -> API -> persistence -> worker -> side effect -> status/feedback.

### Action Inventory

| Area | User Action | UI Entry | API/Handler | State Mutation | Background Owner | External Effect | Visible Result | Unknowns |
|------|-------------|----------|-------------|----------------|------------------|-----------------|----------------|----------|
| ... | ... | ... | ... | ... | ... | ... | ... | ... |

### Lifecycle State Map

| Entity/Flow | States Found | Missing/Overloaded States | Notes |
|-------------|--------------|---------------------------|-------|
| ... | ... | ... | ... |

### Trust Gaps

List findings ordered by severity. Each finding must include evidence and the broken or unproven promise.

Format:

`Severity - Gap title`

Evidence: `path:line` or observed runtime signal.

Broken promise: what the user/system assumes but the code does not prove.

Risk: what can go wrong.

Suggested fix: smallest stabilization step.

### Invariants To Enforce

| Invariant | Why It Matters | Enforce In | Current Coverage |
|-----------|----------------|------------|------------------|
| ... | ... | test/runtime/db constraint/UI | none/partial/covered |

### Risk Matrix

| Rank | Flow | Gap | Severity | Evidence | Test Needed | Suggested Fix |
|------|------|-----|----------|----------|-------------|---------------|
| 1 | ... | ... | Critical/High/Medium/Low | ... | ... | ... |

### Test Matrix

| Flow | Promise Under Test | Test Type | Mocked Side Effects | Expected Failure If Broken |
|------|--------------------|-----------|---------------------|----------------------------|
| ... | ... | vertical/contract/state/e2e | ... | ... |

### Stabilize Vs Rewrite

Give one recommendation: stabilize, replace a bounded module, or rewrite. Explain why the recommendation beats the alternatives based on evidence.

### Next 3 Actions

List exactly three concrete next actions in priority order. Prefer actions that increase proof or visibility before broad refactors.

## Quality Bar

- Be evidence-backed and direct.
- Separate proven facts from inferences.
- Favor vertical slices over broad generic advice.
- Do not hide unknowns; name them and propose the smallest probe to resolve them.
- Do not claim a flow works because a function exists. It works only if the full promise is proven or observable.
- Do not recommend a full rewrite unless a bounded audit shows stabilization cannot restore trust.
- Do not mutate production data or trigger side effects during the audit.
- Prefer adding diagnostics and characterization tests before refactoring.
- Keep the output actionable enough that a follow-up implementation session can start from the first ranked gap.

## Optional Implementation Mode

Only enter implementation mode if the user explicitly asks to fix or implement after the audit.

When implementing, prefer this order:

1. Add read-only diagnostics for critical flows.
2. Add characterization or vertical tests for the broken promise.
3. Make the smallest code change that restores the invariant.
4. Run targeted tests.
5. Report what is now proven and what remains unknown.
