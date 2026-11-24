---
name: review-triage
description: Use when you receive code review feedback (from a human, another agent, a bot, or a PR comment), before changing any code. Sorts each item into fix, ask, or push back by checking it against the actual codebase, then fixes items one at a time with tests, and replies with facts instead of flattery.
---

# Review Triage

**Rule:** review comments are claims to check, not orders to obey. Verify each one against the code before you change anything, and answer with facts, not praise.

## The loop

1. **Read all of it first.** Don't start fixing item 1 before you've seen item 6; items are often related.
2. **Restate each item** in one line as a concrete change. If you can't, it's unclear.
3. **Ask before touching code** if anything is unclear. Ask about all unclear items at once: "Clear on 1, 2, 3, 6. Need detail on 4 and 5 before I start."
4. **Check each item against the codebase** (table below) and sort it: **fix**, **ask**, or **push back**.
5. **Fix in order:** blockers (breakage, security) → quick fixes (typos, imports, naming) → bigger changes (logic, refactors).
6. **One item at a time:** change, run the relevant tests, then move on. Run the full suite at the end.
7. **Reply per item** with what you did or why you didn't.

## Checking an item

| Ask | How to check |
|---|---|
| Is it actually true here? | Read the code it points at; reproduce the bug it claims |
| Would it break something? | Find callers, run the tests, check supported platforms/versions |
| Is there a reason the code is this way? | `git log -p` / `git blame` the lines, look for comments or linked issues |
| Is the thing even used? | Search for callers. Unused code → offer to delete it instead of "doing it properly" |
| Does it clash with the owner's decisions? | Check the spec, plan, or earlier instructions. If it clashes, stop and ask the owner |

Trust depends on the source. Your user's feedback: act once you understand it. External reviewers, bots, other agents: check before acting. If you can't verify something, say so: "Can't confirm this without a device on iOS 15. Investigate, or proceed as suggested?"

## Push back when

- it breaks existing behavior or tests
- it's wrong for this stack, version, or platform
- it adds a feature nobody uses (YAGNI)
- the reviewer is missing context (legacy support, a past decision)
- it conflicts with what the owner already decided

Push back with evidence: the test, the caller, the commit, the version constraint. Ask a specific question. Never be defensive and never stay quiet to keep the peace.

## How to reply

No "You're absolutely right!", "Great catch!", or "Thanks for the feedback!". The fix shows you listened.

```
1. Fixed: null check added in parseOrder() (orders.ts:42). Test added.
2. Not changed: min target is macOS 10.15 and this API needs 13+. Keep legacy path, or drop pre-13 support?
3. Removed: nothing calls /metrics/export (searched the repo). Restore if you need it.
4. Question: "handle errors properly" here means retry, or surface to the user?
```

If your pushback was wrong, say so in one line and fix it: "Checked: you're right, `X` does `Y`. Fixing now." No long apology, no defending.

On GitHub, reply inside the inline comment thread (`gh api repos/{owner}/{repo}/pulls/{pr}/comments/{id}/replies`), not as a new top-level PR comment.
