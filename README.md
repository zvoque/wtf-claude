<div align="center">

# wtf-claude

**A read-only post-mortem skill for [Claude Code](https://claude.com/claude-code).**

*For when Claude touches 47 files and says "done."*

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-skill-d97757.svg)](https://claude.com/claude-code)
[![Read-only](https://img.shields.io/badge/git-read--only-2ea44f.svg)](#why-read-only)

[![CommitCrimes](https://commitcrimes.dev/badge/zvoque.svg)](https://commitcrimes.dev/u/zvoque)

</div>

---

## Overview

AI coding agents move fast. When a session gets messy, you're left digging
through diffs, half-run commands, stray edits, and a cheerful "done" that proves
nothing. `wtf-claude` answers the question that follows:

> **What the f\*\*\* just happened?**

Run `/wtf` after a confusing session, a messy working tree, a failed run, or a
transcript pasted from another agent. It produces a structured, skimmable report:
what the agent tried, what actually changed, what broke, what it guessed, and
exactly what to do next.

It is **read-only**. It diagnoses and hands you copy-paste recipes — it never runs
a destructive git command on your behalf. The moment you're confused is the worst
moment to let an agent take the wheel.

## Installation

Clone into your Claude Code skills directory:

```bash
git clone https://github.com/zvoque/wtf-claude.git ~/.claude/skills/wtf
```

> **Note** — the target directory must be named `wtf`. Claude Code derives the
> slash command from the skill's folder name, so cloning into
> `~/.claude/skills/wtf` is what makes the command `/wtf`. The repository is named
> `wtf-claude`; the installed skill is named `wtf`.

Update later with:

```bash
cd ~/.claude/skills/wtf && git pull
```

## Usage

| Command | Mode | What it does |
| --- | --- | --- |
| `/wtf` | Session | Post-mortems the current session plus live git state. Highest fidelity — the agent already has full context. |
| `/wtf markdown` | Session + archive | Same report, plus writes a timestamped copy to `.wtf/wtf-<timestamp>.md` (and adds `.wtf/` to `.gitignore` first). |
| `/wtf <paste>` | Forensics | Dissects a transcript or diff pasted from another agent, verifying every claim against your actual working tree. |

## Report format

Every report opens with a three-line verdict, so you get the gist in seconds:

```
WTF — quick verdict
  Tried:   what the agent was attempting
  Changed: the headline — N files, the one thing that matters
  Watch:   the single biggest thing to be suspicious of
```

Followed by the **core four**, always present:

1. **Intent** — what the agent was actually trying to do.
2. **What changed** — every file, grouped, each with a one-line reason.
3. **What broke** — failing commands, tests, and builds, quoted exactly.
4. **Safest next prompt** — a ready-to-paste instruction to move forward.

Then sections that appear **only when they have something to say**, so a clean run
stays short and a chaotic one grows only as much as it must:

- **Should have run but didn't** — the verification the agent skipped, named with
  your repo's real commands (read from `package.json`, `Makefile`, or `CLAUDE.md`).
- **Suspicious assumptions** — guesses the agent made and ran with.
- **Risky or unrelated edits** — scope creep and drive-by changes.
- **Rollback recipes** — exact, copy-paste, least-destructive-first. Printed, never run.
- **Human review checklist** — only the calls a human must make.

## Why read-only

When you run `/wtf`, you're already disoriented and possibly about to lose work.
A diagnostic that also starts reverting files turns a confusing session into a
destroyed one. So the skill runs only safe git inspection — `status`, `diff`,
`log`, `stash list` — and prints every destructive action as a recipe for **you**
to run after you've read what it found.

## License

[MIT](LICENSE) © zvoque
