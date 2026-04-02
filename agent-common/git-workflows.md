# Git Workflows

Reference material for git operations. For PR workflows, use the slash commands:
- `/pr-review` - Comprehensive code review
- `/commit` - Generate conventional commit message

---

## Getting the Diff

**IMPORTANT**: Always compare against `origin/main` (not local `main`) and fetch first to ensure you have the latest remote state:

```bash
git fetch origin && git diff origin/main...HEAD
```

**Why `origin/main` instead of `main`?**

- Local `main` may be outdated if not recently pulled
- `origin/main` always reflects the actual remote branch state after fetch
- This ensures accurate PR diffs that match what GitHub will show

---

## Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

### Type Prefixes
- `feat:` - New feature
- `fix:` - Bug fix
- `docs:` - Documentation only
- `refactor:` - Code change that neither fixes a bug nor adds a feature
- `test:` - Adding or updating tests
- `chore:` - Maintenance tasks, dependencies, config

### Subject Line Rules
- **Aim for ≤50 characters** total (including type)
- Maximum 72 characters
- Use imperative mood ("add feature" not "added feature")

### Body
Use for detailed explanations (no line wrapping required).

### Examples
```
feat: add sunrise-based alarm trigger
```

```
fix: resolve spotcast device selection

The wrong device was selected because the entity_id lookup
didn't account for grouped speakers.
```

```
chore: update spotcast to v6.0.0-a16
```

---

## Norwegian Translation Guidelines

When creating Norwegian versions of documentation or PR summaries:

### What to Keep in English
- **Technical terms**: "backend", "commit message", "type safety", "PR review", "source of truth", "conditionals", "callback", "state", "discovery", "runtime"
- **Code concepts**: "interface", "mock", "async/await", "hooks"
- **Tool/framework names**: "Home Assistant", "pyscript", "HACS", "Spotcast"
- **File paths and code snippets**
- **Precision terms**: Use English when Norwegian translation would be ambiguous

### What to Translate
- Explanatory text
- Headings and general descriptions
- Non-technical prose

### Italics Usage
Use italics sparingly, only for special named concepts:
- ✅ Specific named systems or integrations
- ❌ Standard technical vocabulary (no italics)

**Key principle**: Italics signal "this is a special named thing" not "this is English".

### Examples

| English | Norwegian | Note |
|---------|-----------|------|
| automation trigger | automatiseringstrigger | Compound word |
| source of truth | source of truth | Keep entirely in English |
| entity state | entity state | Technical HA term |
| development workflows | arbeidsflyten når vi utvikler | Natural phrasing |

### Style Notes
- Prefer natural Norwegian sentence structure over word-for-word translation
- Use correct Norwegian spellings (e.g., "heng" not "hang", "kommandoer" not "kommandos")
