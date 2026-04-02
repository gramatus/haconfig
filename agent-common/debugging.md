# Debugging Protocol

Reference documentation for structured debugging.

---

## Quick Reference

### Two-Strikes Rule

| Strike | Action |
|--------|--------|
| 0 | State hypothesis, attempt fix |
| 1 | State NEW hypothesis (not variation), attempt fix |
| 2 | **STOP.** Return control to user (see below) |

**Two strikes = mandatory check-in.** You may not attempt a third fix without first:
1. Telling the user what you've tried and what you observed
2. Sharing what you know vs. what you're uncertain about
3. Getting user input on what to investigate next

This is not optional. Do not continue autonomously after two failed fixes.

### Hypothesis Format (required before each fix)

```
**Hypothesis**: [What I believe the root cause is]
**Evidence for**: [What I expect to see if right]
**Evidence against**: [What I would see if wrong]
**Fix attempt #N**: [What I'm about to try]
```

---

## Session Reset

If you've attempted 5+ fixes without progress, or you're revisiting hypotheses already explored, use `/handoff` to create a structured handoff document for a fresh session.
