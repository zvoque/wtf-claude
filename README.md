# wtf-claude

**For when Claude touches 47 files and says "done."**

A [Claude Code](https://claude.com/claude-code) skill that answers the most
common AI-coding question:

> *"What the f\*\*\* just happened?"*

AI coding agents move fast. When a session gets messy, you're left digging
through diffs, half-run commands, random edits, and a cheerful "done" that
proves nothing. `wtf-claude` turns that chaos into a structured, skimmable WTF
report — what the agent tried, what actually changed, what broke, what it
guessed, and exactly what to do next.

It is **read-only**. It diagnoses and hands you copy-paste recipes. It never runs
a destructive git command on your behalf — because the moment you're confused is
the worst moment to let an agent take the wheel.

---

## Install

Clone straight into your Claude Code skills directory:

```bash
git clone https://github.com/zvoque/wtf-claude.git ~/.claude/skills/wtf
```

The target directory must be named `wtf` — Claude Code derives the slash command
from the skill's folder/name, so cloning into `~/.claude/skills/wtf` is what makes
the command `/wtf` (not `/wtf-claude`). The repo is named `wtf-claude`; the
installed skill is named `wtf`.

To update later: `cd ~/.claude/skills/wtf && git pull`.

---

## Usage

### `/wtf`
Post-mortem the **current session** + live git state. Run it right after a
confusing run — the agent already has full context, so this is the
highest-fidelity mode.

### `/wtf markdown`  (or `/wtf md`)
Same report, **plus** writes a timestamped archive to
`.wtf/wtf-YYYY-MM-DD-HHMMSS.md`. The skill adds `.wtf/` to your `.gitignore`
first, so the report never gets committed by accident.

### `/wtf <pasted transcript or diff>`
**Forensics mode.** Paste a transcript or diff from *another* agent (a different
session, Codex, whatever) and it dissects the blob — verifying every claim
against your actual working tree and flagging anything that doesn't match as
*claimed / unverified*.

---

## What you get

Every report leads with a three-line verdict so you get the gist in five seconds:

```
WTF — quick verdict
  Tried:   <what the agent was attempting>
  Changed: <the headline — N files, the one thing that matters>
  Watch:   <the single biggest thing to be suspicious of>
```

Then the **core four**, always present:

1. **Intent** — what the agent was actually trying to do
2. **What changed** — every file, grouped, with a one-line *why*
3. **What broke** — failing commands/tests/builds, quoted exactly
4. **Safest next prompt** — a ready-to-paste instruction to move forward

…followed by sections that appear **only when they have something to say** — so a
clean run is short and a chaotic one is long:

- Commands & tests run
- **Should have run but didn't** — the verification the agent skipped, named with
  your repo's *real* commands (read from `package.json` / `Makefile` / `CLAUDE.md`)
- Suspicious assumptions the agent made and ran with
- Risky or unrelated edits (scope creep, drive-by changes)
- **Rollback recipes** — exact, copy-paste, least-destructive-first (printed, never run)
- Human review checklist — only the spots a human must judge

---

## Why read-only matters

When you run `/wtf`, you're already disoriented and possibly about to lose work.
A diagnostic that *also* starts reverting files is how you turn a confusing
session into a destroyed one. So this skill only ever runs safe, read-only git
inspection (`status`, `diff`, `log`, `stash list`). Every destructive action is
printed as a recipe for **you** to run after you've read what it found.

---

## License

MIT © zvoque
