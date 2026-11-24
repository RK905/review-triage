# review-triage

An agent skill that makes your AI coding agent check every code review comment against the real codebase before changing anything, then fix, ask, or push back, with facts instead of "You're absolutely right!".

Works with Claude Code and any agent harness that loads `SKILL.md` skills.

## Why

Agents handle review feedback badly in two opposite ways: they agree with everything and blindly apply it (breaking things the reviewer didn't know about), or they half-understand a list of comments and fix the wrong half. This skill gives them a short triage routine:

- read all the feedback before touching code, and ask about unclear items up front
- verify each claim against the code, history, and tests
- sort each item into **fix**, **ask**, or **push back**, with evidence
- fix one item at a time, testing as it goes
- reply per item with what changed, not flattery

## How it works

```
  review feedback arrives (human, bot, agent, PR comments)
          │
          ▼
  1. Read all of it ─────── don't start on item 1 yet
          │
          ▼
  2. Restate each item ──── unclear? ask about all of them now
          │
          ▼
  3. Check against code ─── callers, git blame, tests, versions, spec
          │
          ▼
  4. Sort ───────────────── fix │ ask │ push back (with evidence)
          │
          ▼
  5. Fix one at a time ──── blockers → quick fixes → big changes, test each
          │
          ▼
  6. Reply per item ─────── "Fixed: …" / "Not changed: … because …"
```

| Reviewer says | Agent checks | Likely outcome |
|---|---|---|
| "Remove this legacy code" | Supported OS/versions, `git blame` | Push back if still needed |
| "Implement this properly" | Is anything calling it? | Offer to delete if unused (YAGNI) |
| "This will crash on null" | Read the code, write a repro test | Fix with a test |
| "Handle errors better" | Nothing to check yet: too vague | Ask what "better" means |

## Install

**Claude Code (personal, all projects):**

```bash
git clone https://github.com/RK905/review-triage /tmp/rt
cp -r /tmp/rt/skills/review-triage ~/.claude/skills/
```

**One project only:** copy `skills/review-triage` into `<project>/.claude/skills/`.

**Other harnesses:** point your agent at `skills/review-triage/SKILL.md` (Codex, Cursor, Gemini CLI, etc. can load it as an instruction file or rule).

## Usage

The skill triggers on its own when the agent receives review feedback. You can also call it directly:

```
/review-triage address the comments on PR #128
```

| Input | What it is | Example |
|---|---|---|
| `feedback` | The review comments, a PR, or a file of notes | `PR #128`, pasted comments |
| `source` | Who gave it (affects how much to verify) | `teammate`, `CodeRabbit`, `subagent` |
| `constraints` | Decisions the fixes must respect (optional) | `must support iOS 15`, `docs/plan.md` |

**Example reply:**

```
1. Fixed: null check added in parseOrder() (orders.ts:42). Test added.
2. Not changed: min target is macOS 10.15 and this API needs 13+. Keep legacy path, or drop pre-13 support?
3. Removed: nothing calls /metrics/export (searched the repo). Restore if you need it.
4. Question: "handle errors properly" here means retry, or surface to the user?
```

## Tips

- Pair it with [proof-before-done](https://github.com/RK905/proof-before-done) so "Fixed" always comes with a test run.
- Tell the agent who your trusted reviewers are (in `CLAUDE.md` or similar); bot and agent reviewers get checked harder.
- Pushback with evidence is a feature. If the agent's pushback is wrong, it should correct itself in one line and move on.

## Credits

Based on the `receiving-code-review` skill from [obra/superpowers](https://github.com/obra/superpowers) by Jesse Vincent, MIT licensed. See [NOTICE](NOTICE) for what changed.

## License

MIT. See [LICENSE](LICENSE).
