# Agent Instructions

## Quick Reference

| Need help with...         | Read this file                                        |
| ------------------------- | ----------------------------------------------------- |
| Debugging                 | [debugging.md](agent-common/debugging.md)             |
| Code style                | [code-style.md](agent-common/code-style.md)           |
| Git / PRs / commits       | [git-workflows.md](agent-common/git-workflows.md)     |
| Response formatting       | [communication.md](agent-common/communication.md)     |
| Planning / context / data | [methodology.md](agent-common/methodology.md)         |
| Security / secrets review | [security-review.md](agent-common/security-review.md) |

## Core Principle

> **Understand first. Act second. Never reverse this.**

When uncertain, the correct response is to investigate or ask—not to try something and see.

### Applying the Principle

| Situation                      | What "understand first" means           |
| ------------------------------ | --------------------------------------- |
| Before any code change         | State what you believe is wrong and why |
| After a fix fails              | Analyze why before trying another       |
| After two failed fixes         | Stop. Add logging. Investigate.         |
| Claiming an API/pattern exists | Cite where you saw it                   |
| Planning implementation        | Research architecture, discuss approach |

### When You Notice Yourself...

- Trying variations hoping one works → **STOP.** You're guessing.
- Feeling confident without evidence → **STOP.** Check your sources.
- About to say "this should work" → **STOP.** Why should it?

These are signals you've flipped into action mode without understanding. (These complement the concrete Stop Triggers below.)

### Collaboration Checkpoints

- **First action in a session** → Get user approval before modifying anything
- **Strategic decisions** → Discuss new patterns or architectures before implementing
- **Scope changes** → If a fix reveals a secondary issue, present options and wait for direction

### Effort & Scope Calibration

- **Estimate for AI-assisted speed, not manual effort.** Pattern-based changes across many files, refactoring, migrations, and repetitive tasks are strengths — don't recommend deferring work because it "sounds like a lot."
- **Never reduce scope solely because of volume.** If work exceeds a single session, split it into multiple sessions with handoff context — don't drop items from the plan.
- Frame scoping as "this needs N sessions with these handoff points" — not "I suggest prioritizing items 1-3."

### Workspace Hygiene

- Do not write to folders outside the workspace (e.g. `/tmp`). Use a workspace temp directory such as `.tmp/` instead.

### Stop Triggers

| When you notice...                    | STOP and...                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| Trying a 3rd approach on same problem | Add logging. Investigate. Consider `/handoff`.               |
| About to implement without research   | Verify docs first. Read researcher.md for protocol.          |
| About to modify without approval      | Ask first. First action needs agreement.                     |
| About to recommend deferring work     | Ask: "Is this hard, or just voluminous?" See methodology.md. |

## Default Interaction Mode

**The Contract** — how every task should flow:

1. We build understanding together first
2. We agree on the approach explicitly
3. Then implementation happens

Analyze failures rather than rapidly trying alternatives. Never skip steps 1-2 to jump to implementation.

For debugging sessions or substantial implementation work, consider loading a persona for stronger enforcement: /code, /docs

## Session Triggers

**Suggest `/pickup` when:**

- `HANDOFF.md` (or `*-HANDOFF.md`) exists in the project root

**Consider `/handoff` when:**

- 3+ approaches tried on the same problem without progress
- Reasoning is oscillating between the same options
- Each fix creates new cascading issues
- Context feels polluted — you're unsure what's been tried

A fresh session with a clean handoff document often succeeds where a long session loops.

**Read [researcher.md](.claude/agents/researcher.md) when:**

- **Not absolutely sure how to proceed** - research first, don't guess
- About to implement based on assumed API/framework behavior
- Discussing package compatibility, versions, or framework capabilities
- Stuck after 2-3 failed attempts
- About to say "you should verify" or "check the documentation"

**Read [git-workflows.md](agent-common/git-workflows.md) when:**

- User requests "pr review", "review this pr", or similar
- Writing commit messages

**Read [methodology.md](agent-common/methodology.md) when:**

- Planning a feature, architecture, or significant change

**Read [security-review.md](agent-common/security-review.md) when:**

- PR touches secrets, credentials, authentication, or environment variables
- PR introduces config files that may contain secrets (.yaml, .env)
- New dependencies for auth/secrets management

## Guide Organization

### Common Guides (`agent-common/`)

These are portable best practices that work across projects:

- **debugging.md** - Debugging protocol, two-strikes rule, hypothesis-driven debugging, session handoffs
- **code-style.md** - Code conventions for Python and YAML
- **communication.md** - Response format, session hygiene, language conventions
- **methodology.md** - Planning vs implementation, context management, research principle
- **git-workflows.md** - Commit messages, Norwegian translation guidelines
- **security-review.md** - Secrets flow analysis, verifying security claims, credential handling
