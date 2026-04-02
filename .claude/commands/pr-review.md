---
description: Self-review current branch before creating PR
---

# Pre-PR Review

Self-review of the current branch (or a specified branch) before creating a pull request. Uses parallel specialized agents to catch issues before teammates see them.

ultrathink: Take time to deeply analyze rather than pattern-match. The goal is to catch issues before teammates see them — a shallow review defeats the purpose.

## Process

### Phase 1: Gather Context

**IMPORTANT: Do NOT checkout other branches.** Review branches remotely using `<branch>`.

Determine the branch to review and the base to compare against:

**Branch (`[BRANCH]`):**
- If `$ARGUMENTS` contains a branch name → use `<branch>` (only use `origin/<branch>` if asked to do so, except that `main` always should be `origin/main`)
- Otherwise → use `HEAD` (current branch)

**Base (`[BASE]`):**
- Default: `origin/main`
- If the user specifies a different base (e.g., "with test/backend as base") → use that branch instead

Run these commands to understand the change:

```bash
# For current branch (no arguments), default base:
git fetch origin && git diff origin/main...HEAD --stat  # summary
git diff origin/main...HEAD                              # full diff
git log origin/main..HEAD --oneline                      # commits

# For a specified branch (e.g., $ARGUMENTS = "mise-setup"):
git fetch origin && git diff origin/main...mise-setup --stat
git diff origin/main...mise-setup
git log origin/main..mise-setup --oneline

# For a custom base (e.g., base = "test/backend", branch = "test/frontend"):
git fetch origin && git diff test/backend...test/frontend --stat
git diff test/backend...test/frontend
git log test/backend..test/frontend --oneline
```

Throughout this review, replace `[BASE]` and `[BRANCH]` in all commands and agent prompts with the actual branch names determined above.

Determine which reviews apply based on files changed:
- **Code review**: Always runs
- **Security review**: Triggers if changes touch secrets, auth, env vars, `.toml`/`.yaml`/`.env` patterns, or make security claims
- **Documentation review**: Triggers if changes touch `docs/**`, `README.md`, or setup guides (human-facing docs)
- **Agent instruction review**: Triggers if changes touch `CLAUDE.md`, `.claude/agents/*`, or `agent-common/*`
- **Test quality review**: Triggers if changes include test files (detected by naming/path patterns from Phase 1.5)
- **Behavioral equivalence review**: Triggers if the diff contains structural changes expected to preserve behavior — refactors, type fixes, renames, bulk cleanups. Detect by: branch/commit messages containing `refactor`/`rename`/`restructure`/`cleanup`/`type-fix`/`migrate`, file renames in the diff (`git diff --diff-filter=R`), or widespread similar changes across many files (e.g., same type annotation fix repeated). If unsure, err toward triggering — a false positive is cheap, a missed behavioral change is not.

---

### Phase 1.5: Assess PR Size and Plan Review Coverage

After gathering the diff stat, determine the review strategy. Each triggered review type independently decides whether it needs batching based on its own file count.

**Step 1: Count changed files**

```bash
# Count files that need review (exclude docs/agent-instructions handled by other agents)
git diff [BASE]...[BRANCH] --name-only \
  | grep -viE '(^docs/|CLAUDE\.md|\.claude/agents/|agent-common/)' \
  | wc -l
```

**If ≤25 reviewable files:** Use a single agent per triggered review type (standard flow). Skip to Phase 2.

**If >25 reviewable files:** Enter **Large PR Mode** — continue with Steps 2-4.

**Step 2: Classify files**

Separate changed files into two categories using language-agnostic heuristics:

```bash
# Test files — by naming convention OR dedicated test directory
git diff [BASE]...[BRANCH] --name-only | grep -iE \
  '\.(test|spec)\.[^.]+$|_test\.[^.]+$|_spec\.[^.]+$|Tests\.[^.]+$|Test\.[^.]+$|^tests/|^test/|__tests__/|^spec/'

# Production code — everything else (minus docs/agent-instructions already handled)
# = total reviewable files minus the test files above
```

When unsure if a file is test or production, classify it as **production** (higher-priority review).

**Step 3: Count files per triggered review type**

For each triggered review from Phase 1, count how many files fall within its scope. A file can belong to multiple review scopes (e.g., a component with type fixes counts for both code review and equivalence review).

| Review type | Scope |
|-------------|-------|
| Code review | All changed code files |
| Behavioral equivalence | Files with structural changes (type fixes, renames, import changes, bulk cleanups) |
| Security | Files touching auth, credentials, env vars, secrets |
| Test quality | Test files |
| Docs / Agent instructions | Typically small — always single agent |

**Step 4: Build batch assignments per review type**

Each triggered review type independently decides whether to batch:

| Files in scope | Strategy |
|---------------|----------|
| ≤15 files | Single agent (standard template) |
| 16+ files | Batch using rules below |

Batching rules:
1. **Production code** gets its own dedicated batch(es). If production files exceed 30, split by top-level directory.
2. **Test files** are split into batches of **15-25 files** each (or **10-15** for equivalence review, which reads both versions per file).
3. **Per review type:** target ≤5 agents. If that would require >35 files per batch, add more (up to ~8 per type).
4. **Total agent budget: ~12.** If the sum across all review types exceeds the budget, reduce batch counts proportionally — prioritize review types with more files in scope.
5. **Empty categories** → skip (no agent for zero files).
6. **Record the batch plan** (per-type agent counts and file assignments) before proceeding to Phase 2.

If the total exceeds ~12 even after proportional reduction, warn the user: _"This PR has [N] files across [M] review concerns — consider splitting into smaller PRs. Proceeding with best-effort coverage."_

---

### Phase 2: Parallel Focused Reviews

Spawn subagents in parallel using the Task tool. **All applicable reviews MUST run in a single message with multiple Task calls.**

**IMPORTANT: Do NOT include the diff in the prompt.** Each agent fetches the diff themselves.

#### Batching: Standard vs Batched Templates

Each review type below has a **standard template** (single agent) and a **batched template** (multiple agents with file assignments). Use the batched template when Phase 1.5 Step 4 assigned multiple agents to that review type.

All agents — batched or not — MUST be spawned in a single message.

#### Code Review Agent(s)

**Standard path** (≤15 files in scope) — spawn a single code review agent:

```
Task(subagent_type="code-reviewer", prompt="""
Review the [BRANCH] branch for code quality. Focus ONLY on code — other agents handle security and docs.

Get the diff:
```bash
git fetch origin
git diff [BASE]...[BRANCH] --stat
git diff [BASE]...[BRANCH]
```

To read a file: use the Read tool with the file path.
""")
```

**Batched path** (16+ files in scope) — spawn one agent per batch from Phase 1.5:

```
Task(subagent_type="code-reviewer", prompt="""
Review the [BRANCH] branch for code quality. You are Code Review Agent [N] of [TOTAL].
Focus ONLY on code — other agents handle security and docs.

**YOUR ASSIGNED FILES — you MUST review ALL of them:**
[explicit newline-separated file list from Phase 1.5]

**Scope rules:**
- You MUST read and review every file listed above. Do not skip files.
- Do NOT review files outside your list — other agents cover those.
- For each file: use the Read tool to read the full file, then review the diff changes in context.
- Focus on: correctness, test quality (meaningful assertions, edge cases), patterns, maintainability.

Get the diffs for your assigned files only:
```bash
git fetch origin
git diff [BASE]...[BRANCH] -- [space-separated file list]
```

To read a file: use the Read tool with the file path.

**At the end of your review, include a coverage report:**
```
## Coverage
Assigned: [N] files
Reviewed: [N] files
Skipped: [list any skipped files with reason, or "None"]
```
""")
```

#### Security Review Agent

Only spawn if security-relevant changes detected.

```
Task(subagent_type="security-reviewer", prompt="""
Review the [BRANCH] branch for security issues. Focus ONLY on security — other agents handle code quality and docs.

Get the diff:
```bash
git fetch origin
git diff [BASE]...[BRANCH] --stat
git diff [BASE]...[BRANCH]
```

To read a file: use the Read tool with the file path.
""")
```

#### Documentation Review Agent

Only spawn if human-facing documentation changes detected (`docs/**`, `README.md`, setup guides).

```
Task(subagent_type="doc-reviewer", prompt="""
Review the [BRANCH] branch for documentation quality. Focus ONLY on human-facing docs — other agents handle code, security, and agent instructions.

Get doc changes:
```bash
git fetch origin
git diff [BASE]...[BRANCH] --stat | grep -E 'README|docs/'
git diff [BASE]...[BRANCH] -- 'README.md' 'docs/'
```

To read a file: use the Read tool with the file path.
""")
```

#### Test Quality Review Agent

Only spawn if test files detected in the diff (using the naming/path patterns from Phase 1.5). Pass the test file list explicitly.

```
Task(subagent_type="test-quality-reviewer", prompt="""
Review test files on the [BRANCH] branch for test quality. Focus ONLY on whether tests verify meaningful behavior — other agents handle code quality and style.

**YOUR ASSIGNED TEST FILES:**
[explicit newline-separated test file list from Phase 1.5]

Get the diffs for your assigned files:
```bash
git fetch origin
git diff [BASE]...[BRANCH] -- [space-separated test file list]
```

For each test file, also read its corresponding production code using the Read tool — you need both to evaluate test quality.
""")
```

#### Behavioral Equivalence Review Agent(s)

Only spawn if structural/behavioral-equivalence-sensitive changes detected (see trigger criteria in Phase 1).

**Standard path** (≤15 files in scope) — spawn a single equivalence agent:

```
Task(subagent_type="refactor-reviewer", prompt="""
Review the [BRANCH] branch for unintended behavioral changes hidden in structural modifications. Focus ONLY on behavioral equivalence — other agents handle code quality, security, and docs.

Get the diff:
```bash
git fetch origin
git diff [BASE]...[BRANCH] --stat
git diff [BASE]...[BRANCH]
```

To read a file on the current branch: use the Read tool.
To read the main version for comparison:
```bash
git show [BASE]:<filepath>
```

Context: This PR contains structural changes (refactors, type fixes, renames, or bulk cleanups) that should preserve runtime behavior. Your job is to verify that behavior is actually preserved.
""")
```

**Batched path** (16+ files in scope) — spawn one agent per batch from Phase 1.5. Use smaller batches (10-15 files) since equivalence review reads both versions of every file and traces consumers.

```
Task(subagent_type="refactor-reviewer", prompt="""
Review the [BRANCH] branch for unintended behavioral changes. You are Equivalence Review Agent [N] of [TOTAL].
Focus ONLY on behavioral equivalence — other agents handle code quality, security, and docs.

**YOUR ASSIGNED FILES — you MUST review ALL of them:**
[explicit newline-separated file list from Phase 1.5]

**Scope rules:**
- You MUST review every file listed above. Do not skip files.
- Do NOT review files outside your list — other agents cover those.
- For each file: read the current version (Read tool) AND the base version (`git show [BASE]:<filepath>`), then compare for behavioral differences.
- After reviewing your files, check for orphaned consumers: files NOT in your list that import from your assigned files.

Get the diffs for your assigned files only:
```bash
git fetch origin
git diff [BASE]...[BRANCH] -- [space-separated file list]
```

Context: This PR contains structural changes that should preserve runtime behavior. Your job is to verify equivalence for your assigned files.

**At the end of your review, include a coverage report:**
```
## Coverage
Assigned: [N] files
Reviewed: [N] files
Skipped: [list any skipped files with reason, or "None"]
Orphaned consumers checked: [N] files
```
""")
```

#### Agent Instruction Review Agent

Only spawn if agent instruction files changed (`CLAUDE.md`, `.claude/agents/*`, `agent-common/*`).

```
Task(subagent_type="agent-instruction-reviewer", prompt="""
Review the [BRANCH] branch for agent instruction quality. Focus ONLY on AI agent instructions — other agents handle human-facing docs.

Get instruction file changes:
```bash
git fetch origin
git diff [BASE]...[BRANCH] --stat | grep -E 'CLAUDE\.md|\.claude/agents/|agent-common/'
git diff [BASE]...[BRANCH] -- 'CLAUDE.md' '.claude/agents/*.md' 'agent-common/*.md'
```

To read a file: use the Read tool with the file path.
""")
```

---

### Phase 3: Devil's Advocate

After collecting subagent results, actively try to break the combined review:

- **Cross-domain issues** — Did security miss something the code review found? Did docs describe something code doesn't do?
- **Gaps between agents** — What might have fallen through the cracks because each agent focused on their domain?
- **Coverage gaps** (Large PR Mode) — Check each batched agent's coverage report, across all review types. If any agent skipped files or reviewed fewer than assigned, flag this explicitly and consider whether a follow-up agent is needed.
- **Boundary issues** (Large PR Mode) — Look for cross-file issues that no single agent could see because files were in different batches: imports/exports across batch boundaries, shared types modified in one batch and consumed in another, test files that test production code reviewed by a different agent.
- **Test quality vs code review alignment** — Did code review flag test coverage gaps that the test quality reviewer also caught from a different angle? Did test quality review find issues in tests that code review praised?
- **Equivalence vs code review overlap** — Did the behavioral equivalence reviewer flag something the code reviewer missed because they were focused on quality? Did code review approve a "cleanup" that the equivalence reviewer identified as a behavioral change?
- **Assume you missed something** — Re-scan the diff for anything no agent covered

If you find issues, add them to the final output.

---

### Phase 4: Synthesize Results

#### Step 1: Completeness Check

You MUST include every issue from every subagent. Common violations:
- ❌ Consolidating "similar" issues into one (they're separate findings)
- ❌ Downgrading severity without factual evidence the agent was wrong
- ❌ Omitting issues that seem minor
- ❌ Summarizing instead of listing

**Verification step:** Before writing the final review, count issues from each agent that ran:
- Code review agent 1: ___ Blocking, ___ Should Fix, ___ Consider
- Code review agent 2: ___ Blocking, ___ Should Fix, ___ Consider (if Large PR Mode)
- [... repeat for each code review agent]
- Security review: ___ Blocking, ___ Should Fix, ___ Consider
- Behavioral equivalence review: ___ Blocking, ___ Should Fix, ___ Consider
- Doc review: ___ Blocking, ___ Should Fix, ___ Consider
- Agent instruction review: ___ Blocking, ___ Should Fix, ___ Consider
- Test quality review: ___ Blocking, ___ Should Fix, ___ Consider

Your final output MUST have the same total count (after deduplication if multiple code agents flagged the same cross-cutting issue).

#### Step 2: Filter False Positives

Mark (don't drop) findings that are false positives:

- **Claude Code built-ins**: `WebSearch`, `Task`, agent frontmatter are Claude Code features, not missing project docs
- **Intentional patterns**: Dev-only scripts don't need production-grade error handling
- **Already mitigated**: If `.gitignore` covers sensitive files, note the mitigation

Format: "[FILTERED: reason] Original finding text"

#### Step 3: Triage by Impact

Categorize remaining findings:

| Priority | Category | Examples |
|----------|----------|----------|
| **P1** | User/Security Impact | Data loss, security holes, breaks user workflows, PHI/PII exposure |
| **P2** | Correctness | Wrong behavior, broken functionality, missing error handling |
| **P3** | Maintainability | Confusing code, missing project-specific docs |
| **P4** | Code Hygiene | Style, unused variables, arbitrary constants in dev-only code |

#### Step 4: Structure for Prominence

**Lead with what matters.** P1/P2 issues must be immediately visible:
- Critical section comes FIRST with full context
- P3/P4 findings in a condensed "Other Findings" section
- Filtered items listed at the end for transparency

**Number every finding sequentially** — Assign each finding a number (#1, #2, #3...) when first listed in the body sections. These same numbers MUST appear in the Findings Summary table. Never list a finding without its number, and never renumber between sections.

When synthesizing:
- **Issues**: Group by priority tier, not by source agent
- **Strengths**: Only include if the agent provided evidence. Remove vague praise.

**BEFORE writing "What's Good"** — scan ALL agent outputs for:
- Any finding marked OVERSTATED or FALSE
- Any claim flagged as not matching implementation
- Any "Consider" item that questions a feature's existence

If an agent questioned it, do NOT praise it.

**Zero or minimal praise is valid.** Don't invent praise to be nice. If the PR has few genuine strengths, say so honestly. Don't invent praise to be nice — that undermines the review's credibility. A PR with blocking issues and no clear strengths should have a short or empty "What's Good" section. The goal is accuracy, not feelings. We want to reinforce good things by being clear that they are good, but we don't want to encourage bad practices by praising them.

Produce the final review:

```markdown
## PR Review: [branch-name]

### Overview
Brief description of what this branch accomplishes.

### Critical Issues (address before merge)

[P1/P2 issues with full context. If none, write "None found."]

- **#1 [Category]**: [file:line](path#L123) - Description with context
- **#2 [Category]**: [file:line](path#L123) - Description with context

### What's Good (✅)
Factual strengths only — backed by evidence from agent reviews.

### Other Findings

**Should address:**
- **#3** [file:line](path#L123) - Description
- **#4** [file:line](path#L123) - Description

**Minor / consider:**
- **#5** [file:line](path#L123) - Description

**Filtered** (included for completeness, marked as false positives):
- **#6** [FILTERED: Claude Code built-in] Finding text
- **#7** [FILTERED: dev-only tooling] Finding text

**Total findings: X** (matches subagent count: code-1=A, code-2=B, ..., security=C, equivalence=D, docs=E, agent-instruction=F, test-quality=G)

### Risk Assessment
Low/Medium/High with reasoning.

### Verdict
**Ready to submit** / **Address critical issues first** / **Needs discussion: [topic]**

### Findings Summary

All findings numbered for easy cross-referencing when fixing issues.

**P1 — Critical:**

| # | Location | Issue | Status |
|---|----------|-------|--------|
| 1 | file.ts:123 | Brief description | |

**P2 — Correctness:**

| # | Location | Issue | Status |
|---|----------|-------|--------|
| 2 | other.ts:456 | Brief description | |

**P3 — Should Address:**

| # | Location | Issue | Status |
|---|----------|-------|--------|
| 3 | file.ts:100 | Brief description | |
| 4 | file.ts:101 | Brief description | |

**P4 — Minor/Consider:**

| # | Location | Issue | Status |
|---|----------|-------|--------|
| 5 | file.ts:200 | Brief description | |

**Filtered (not actionable):**

| # | Location | Issue | Status |
|---|----------|-------|--------|
| 6 | file.ts:300 | Reason it's filtered | |
| 7 | file.ts:400 | Reason it's filtered | |

**Total: X findings** (P1: A, P2: B, P3: C, P4: D, Filtered: E) — **Y fixed, Z remaining**

> **Workflow:** User marks intent → Agent marks completion
> - 🎯 = fix this → ✅ when done
> - 🤔 = discuss first
> - ⏭️ = skip
```

#### Step 5: Findings Summary Table

After the main review, **always** include a numbered summary table grouped by priority:

1. **Include workflow hint** — Add the blockquote explaining the two-phase workflow (user intent → agent completion)
2. **Separate table per priority** — P1, P2, P3, P4, and Filtered each get their own table with header. **Omit** priority sections that have zero findings (don't show an empty table).
3. **Number sequentially across all tables** — Numbers continue across priority levels (1, 2, 3... not restarting)
4. **Include Status column** — User marks intent (🎯/🤔/⏭️), agent adds ✅ when fix is implemented
5. **Keep descriptions brief** — One-line summary, details are in the main review
6. **Add totals** — Show counts by priority and fixed/remaining summary

**Two-phase workflow:**
- **Phase 1 (User):** Reviews findings and marks each with intent: 🎯 (fix), 🤔 (discuss), ⏭️ (skip)
- **Phase 2 (Agent):** Works through 🎯 items, adding ✅ to Status column as each fix is implemented

**Updating the review document:** When fixes are complete, update the findings table in the review document (e.g., `.claude/sessions/.../pr-review.md`) to reflect the final status. Change 🎯 → ✅ for completed items and update the totals line.

This table serves as a checklist for agentic fix sessions. Users can reference issues by number (e.g., "Fix #3 and #7").

---

## Review Principles

- **Full coverage** — Every changed file must be reviewed by at least one agent. For large PRs, split across multiple code review agents with explicit file assignments. Spot-checking is not a review.
- **Parallel execution** — Spawn all applicable subagents in a single message
- **Focused expertise** — Each agent owns one domain; don't duplicate
- **Completeness first** — Include all findings, then triage for prominence
- **Mark, don't drop** — False positives are filtered but still listed
- **P1/P2 prominent** — Critical issues must be unmissable
- **No vague strengths** — Only include what agents demonstrated with evidence
- **Devil's advocate everywhere** — Each agent AND synthesis must try to break their findings
- **Numbered summary table** — Always end with a numbered table for easy cross-referencing

## After Review

Remind user:
- "Consider running `/bookkeeping` to review active work context and instruction set health."


ARGUMENTS: $ARGUMENTS
