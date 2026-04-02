# Methodology

How to approach work, manage context, and ensure correctness.

## Planning vs Implementation

**During planning and design discussions:**

- **Focus on the broad picture**: Architecture, file structure, dependencies
- **Avoid detailed code snippets** unless specifically requested or needed for critical clarity
- **Use short examples** (2-5 lines) only when they significantly enhance understanding
- **Keep it scannable**: Use bullet points, tables, and clear headings

**When user requests implementation or asks for code details:**

- **Then provide full code snippets** with proper context
- **Show complete diffs** for file changes
- **Provide runnable examples** that can be directly applied

**Key principle**: Planning should help the user understand the approach and make decisions without overwhelming them with implementation details they haven't requested yet.

---

## Scoping & Session Planning

When a task involves significant volume (many files, large migration, multi-phase refactor):

**Do:**
- Plan the full scope, then divide into session-sized chunks
- Define clear handoff points between chunks

**Don't:**
- Estimate effort in human-hours ("this would take 40-80 hours")
- Recommend deferring work items because the total list is long
- Say "prioritize items 1-3, defer 4-7" — instead say "session 1: items 1-3, session 2: items 4-7"
- Treat file count as a complexity signal — touching 100 files with a consistent pattern is routine

**The test:** If you're about to recommend skipping or deferring work, ask: "Is this actually hard, or just voluminous?" Volume is not difficulty.

---

## Context Management

- Use attached context or explicit mentions (`#file`, `#selection`, `#path`)
- **Include all relevant context** for high-quality answers; expand as needed
- **Confirm before adding unrelated files** (files outside the current task scope or feature area)
- If context appears stale, ask to refresh specific files rather than proceeding
- When uncertain about relevance, explain why additional context might help and ask for permission

---

## Verify, Don't Defer

**Never recommend that the user verify something you can check yourself.**

Anti-patterns to avoid:
- "Ensure X is configured correctly" → Check the config file and state whether it is
- "Verify that Y exists" → Use Glob/Grep/Read to confirm it exists
- "You should check if Z" → Check it yourself and report the finding
- "Consider adding X to .gitignore" → Read .gitignore and confirm whether X is already there

**The principle**: If you have the tools and data to answer a question, answer it. State facts, not recommendations to investigate. This applies to:
- File existence and contents
- Configuration values
- Whether something is already implemented
- Whether a pattern is already followed

The only exceptions are runtime verification (actual behavior testing) or access to systems you genuinely cannot reach.

---

## Research

**Primary enforcement:** `researcher` subagent (auto-triggered in Claude Code)

**Principle:** Research by default. Only skip if you can cite documentation read this session. "I'm confident" is not evidence — training-data confidence is the leading cause of wrong implementations.
