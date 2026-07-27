---
name: logging-and-commits
description: Use this skill whenever working on any multi-step coding task in a code project — implementing features, refactoring, fixing bugs, or any session that will involve more than a trivial one-line change (initialize git first if the project isn't yet a repo). It reminds Claude to commit at natural checkpoints with clear messages, and to maintain a per-session log capturing what was done and the token/cost figures for that session. This operates at the level of a single working session; for an end-of-day roll-up across all of the day's sessions, use the daily-wrap-up skill instead. Trigger this even when the user hasn't explicitly asked for commits or logging, since the goal is to make these habits automatic across the org's Claude Code usage.
---

# Session Logging and Commits

Two habits that make Claude Code sessions safe, reviewable, and cost-aware: commit early and often, and keep a running log of the session including token and cost figures.

## Why this matters

Frequent commits give the user checkpoints to roll back to, make changes reviewable in small pieces, and mean an interrupted or derailed session loses almost nothing. A session log gives the team a paper trail of what an AI-assisted session actually did, and — importantly for a pilot — visibility into how much these sessions cost in tokens and dollars, which is exactly the kind of data an org needs to evaluate the tool. Doing both by default, rather than only when asked, is what turns them into team habits.

## Committing frequently

Commit at every natural checkpoint, not just at the end:

- After completing a discrete, working unit of change (a function, a component, a passing test, a fixed bug).
- Before starting a substantially different piece of work in the same session.
- Before any large or risky refactor, so there's a clean point to return to.
- Whenever the working tree represents a coherent, non-broken state worth preserving.

Guidelines:

- **Keep commits small and focused.** One logical change per commit is easier to review and revert than one giant end-of-session commit.
- **Write clear messages** in conventional-commit style so history is scannable. Format: `type(scope): summary`, e.g. `feat(auth): add password reset flow`, `fix(parser): handle empty input`, `refactor(api): extract request builder`, `test(cart): cover discount edge cases`, `docs(readme): document env vars`.
- **Never commit broken code** knowingly. If mid-change and the tree is broken, finish to a working state (or stash) before committing.
- **Confirm before pushing.** Committing locally is safe and cheap; pushing to a shared/remote branch is a shared-state action — ask the user before `git push`. Same for anything destructive (`git reset --hard`, force-push, history rewrites): surface it and confirm first.
- **Respect the repo's conventions.** If the project has commit hooks, a commit template, or a branching model, follow it.

A good rhythm: make a coherent change → verify it works → commit it → move on. If you notice several unrelated changes piling up uncommitted, that's the signal to commit before continuing.

## Session logging (including token usage and cost)

Maintain a running log for each working session. Append to a project file such as `CLAUDE_SESSIONS.md` (or `docs/sessions/` — match whatever the repo already uses; create `CLAUDE_SESSIONS.md` at repo root if nothing exists).

For each session, record:

- **Date and a short session title / goal.**
- **What was done** — a concise bullet summary of the changes made and decisions taken.
- **Commits made** during the session (short hashes + summaries once committed).
- **Token usage and cost** for the session (see below on how to get accurate figures).
- **Follow-ups / open items** left for next time.

### Getting accurate token and cost figures — important

Claude Code cannot reliably read its own live token count or dollar cost from inside the running session — that accounting lives in the CLI, not in the model's context. **Do not estimate or invent these numbers.** Instead, get the real figures:

- Ask the user to run the **`/usage`** command in Claude Code (its aliases `/cost` and `/stats` do the same thing) and paste the result — or tell you the values so you can record them. `/usage` reports the current session's token usage plus, on subscription plans, how recent usage maps to skills, subagents, plugins, and MCP servers — useful visibility for a pilot.
- Alternatively, at session end the CLI prints a usage/cost summary; the user can copy those figures into the log.
- The dollar figure `/usage` shows is an estimate computed locally from token counts and may differ from the actual bill. For authoritative billing, the source of truth is the Usage page in the Claude Console, not the session estimate.
- If the user is on a flat-rate plan (e.g. a Max/Pro subscription), per-session dollar cost isn't relevant for billing since usage is included in the subscription. Record token counts from `/usage` and note that dollar cost is covered under the subscription rather than fabricating a dollar figure.

Record the figures verbatim as provided. If they aren't available yet, leave a clearly marked placeholder (e.g. `tokens: TBD — run /usage`) rather than guessing.

### Session log template

Use this structure for each entry:

```markdown
## <YYYY-MM-DD> — <Session title>

**Goal:** <one line>

**Changes:**
- <what was done>
- <what was done>

**Commits:**
- `<hash>` <message>
- `<hash>` <message>

**Usage & cost:** <from `/usage` or CLI summary — e.g. "1,240,000 tokens, ~$3.72 (est.)"; or "tokens: TBD — run /usage">

**Follow-ups:**
- <open item>
```

Update the log incrementally as the session progresses (don't wait until the very end and try to reconstruct it), and commit the log file along with the work — the log is part of the record.

## Seeding these habits into CLAUDE.md

This skill is also intended to be reflected in a project's `CLAUDE.md` so the behavior persists for everyone, not just when the skill happens to trigger. When setting up or updating a repo's `CLAUDE.md`, include guidance like:

```markdown
## Commits & session logging

- Commit frequently at natural checkpoints, not just at session end. Keep commits
  small and focused, one logical change each. Use conventional-commit messages
  (`type(scope): summary`). Never knowingly commit broken code. Ask before pushing
  to shared branches or running destructive git commands.
- Maintain a session log in `CLAUDE_SESSIONS.md`: date, goal, summary of changes,
  commits made, and token/cost figures. Get token/cost numbers from the `/usage`
  command or the CLI's end-of-session summary — never estimate them. The dollar
  figure is a local estimate; authoritative billing is in the Claude Console.
  Update the log as you go and commit it with the work.
```

Offer to add this to `CLAUDE.md` when helping set up a repository for the team, adjusting wording to fit the project's existing conventions.
