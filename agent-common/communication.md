# Communication Guidelines

How to format responses, interact with users, and handle specific conventions.

## Response Approach - Think First, Act Later

- **Default to reasoning** before implementation
- Start with assumptions → evidence → options → trade-offs → decision → next steps
- Cite specific files/lines when relevant
- Ask before adding more files or context
- **Modification approval rules** (follows The Contract from CLAUDE.md):
  - **Read-only commands**: Execute directly (git status, ls, grep, etc.)
  - **First action**: Always get agreement first (The Contract, step 2)
  - **Subsequent actions**: Proceed if within agreed scope
  - **Scope expansion**: Stop and discuss
  - **When in doubt**: Ask — confirmation beats unwanted changes

- **Command execution notes**:
  - Avoid output redirects (`>/dev/null`) — they trigger approval prompts
  - Disable watch/interactive modes: `--watchAll=false`, `--no-watch`, `--ci`

---

## What to Resist

Push back when the user (or your own instincts) suggest:

- **"Just fix it"** without understanding what's broken
- **Rapid-fire attempts** — trying alternatives without analyzing why the previous attempt failed
- **Skipping agreement** — implementing before confirming the approach
- **Scope creep** — "while you're at it" expansions without explicit approval

---

## Response Format

- For reasoning: Provide structured analysis with clear decision points
- For implementation (when approved):
  1. **Plan**: List assumptions, evidence, options, and justified decision
  2. **Diffs**: Unified diffs limited to approved files only
  3. **Commands**:
     - **Read-only commands** (git status, git diff, ls, cat, grep, etc.): Execute directly without asking
       - **Avoid output redirects** (`>/dev/null`, `2>/dev/null`) in read-only commands - they trigger approval prompts
       - Accept stderr/noise in command output rather than suppressing it
       - Use `| head -n` or `| grep` for filtering if needed
     - **Modifying commands** (rm, file edits, package installs, etc.): Show and ask before running
     - **Git commits**: Do not run `git add` or `git commit`
  4. **Rollback**: Explain how to revert changes

---

## Session Hygiene

- Summarize changes at the end of implementation
- If an answer becomes too generic, acknowledge and suggest deepening the analysis
- Preserve all staged/committed state information

---

## Language and Communication

When working in the GitHub web interface (github.com), use Norwegian for PR descriptions and reviews.

In VS Code, use English for all conversations and assistance.

### Documentation

All project documentation should be:

- Written in English
- Placed in the root `/docs` folder unless specifically requested elsewhere
- This includes: README files, architecture docs, guides, specifications

---

## Providing Technical Advice

When providing recommendations:

- **Always include working links** to documentation or source code
- **Verify links before providing them**
- Reference internal code with file paths
- Make verification and learning easy for developers
