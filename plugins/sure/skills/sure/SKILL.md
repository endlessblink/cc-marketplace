---
name: sure
description: Confidence gate before implementing a fix or feature.
---

# Sure? — Confidence Gate

Stop and verify your understanding before writing any code.

**Use this skill when:** You want to challenge Claude's confidence before it starts implementing a fix or feature. Invoke with `/sure?` at any point during debugging or planning.

---

## Instructions

Before proceeding with ANY fix or implementation, answer ALL 5 points below. Do NOT start coding until every point is addressed.

### 1. Root Cause
State the root cause in ONE sentence. Include `file:line` if known.

### 2. Confidence Rating
Rate your confidence honestly:
- **HIGH** — I've seen this exact pattern before and know the fix will work
- **MEDIUM** — Strong hypothesis, but I need to verify something first
- **LOW** — I'm guessing or working from incomplete information

### 3. If Not HIGH
What would you need to read, grep, or test before you're confident? List the specific files, functions, or commands.

If LOW: **STOP HERE.** Do the research first, then re-run `/sure?`.

### 4. The Fix
Describe the change in 1-3 bullets:
- What changes
- Where (file paths)
- How it fixes the root cause

### 5. Side Effects & Regressions
List anything that could break:
- Other components consuming the same data/API
- CSS changes that might affect other views
- State mutations that other watchers depend on
- If "none" — explain why you're confident there are no side effects

---

**Format your response as a numbered list matching these 5 points. Be brutally honest — "I don't know" is better than a wrong fix.**
