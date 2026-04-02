---
description: Generate conventional commit message from staged changes
---

# Generate Commit Message

Analyze **only staged changes** and generate a conventional commit message.

## Scope — Read Only

- **ONLY** read `git diff --cached` — that is the commit.
- **NEVER** stage files, unstage files, or run `git add`/`git commit`.
- Base the commit message **only** on what is staged. Never include unstaged changes in the message.
- You may note potentially related unstaged changes, but do not assume they belong in this commit.

## Process

1. **Get the staged diff**:
   ```bash
   git diff --cached
   ```

2. **Analyze the staged changes**:
   - What files are modified/added/deleted?
   - What is the nature of the change? (feature, fix, refactor, docs, test, chore)
   - What is the purpose/intent of the change?

3. **Generate commit message** following conventions:

## Commit Message Format

```
type(scope): subject

[optional body]

[optional footer]
```

### Type Prefixes
- `feat:` - New feature
- `fix:` - Bug fix
- `docs:` - Documentation only
- `refactor:` - Code change that neither fixes a bug nor adds a feature
- `test:` - Adding or updating tests
- `chore:` - Maintenance tasks, dependencies, config

### Subject Line Rules
- **Aim for ≤50 characters** total (including type and scope)
- Maximum 72 characters
- Use imperative mood ("add feature" not "added feature")
- No period at the end

### Body (if needed)
- Explain **why**, not **what** (the diff shows what)
- No line wrapping required

## Examples

```
feat: add sunrise-based alarm trigger
```

```
fix: resolve spotcast device selection

The wrong device was selected because the entity_id lookup
didn't account for grouped speakers.
```

```
refactor: simplify authentication flow

Consolidated three separate auth checks into a single middleware.
This reduces code duplication and makes the flow easier to follow.
```

## Output

Present the suggested commit message and ask if the user wants to:
1. Use it as-is
2. Modify it
