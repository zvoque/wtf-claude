---
name: wtf
description: >-
  Produce a structured "what the f*** just happened" post-mortem of a confusing
  coding session, a messy git working tree, a failed run, or a transcript/diff
  pasted from another agent. Use this whenever the user runs /wtf, or says things
  like "what did you just change", "what just broke", "wtf happened", "explain
  this session", "explain this diff", "what did Claude/the agent actually do",
  "did you break anything", "can I trust this change", or asks for a recap,
  audit, or post-mortem of recent agent work. Trigger even when the user is just
  venting confusion about a chaotic run and hasn't named a format — they want a
  clear account of what changed, what broke, what was guessed, and what to do
  next. Read-only: it diagnoses and suggests, it never runs destructive git.
---

# wtf-claude

The job: turn a confusing coding session into a clear account of **what
happened**. The user is disoriented — an agent (you, or another tool) touched a
bunch of files and said "done", and now they can't tell what's real, what broke,
and whether they can trust it. Give them a brutally clear, skimmable report.

This skill is **read-only and diagnostic**. You gather context, reason about it,
and hand back copy-paste recipes. You never run a mutating git command on the
user's behalf — that's how people lose work mid-panic. See "Hard safety rule".

## Pick the mode (do this first)

Look at what the user passed after `/wtf` (or what they're asking):

1. **No argument** → *session mode*. Analyze the current conversation you're
   already in (the turns above this one) plus the live git state. This is the
   highest-fidelity case — you have full context for free.
2. **First word is `markdown` or `md`** → *session mode + write a file*. Strip
   that word, then analyze exactly as session mode, and also write the report to
   disk (see "Writing the file").
3. **Anything else substantial** (a pasted transcript, log, or diff) →
   *forensics mode*. Dissect the pasted blob as the primary source. Use git
   state only to corroborate — the paste may describe a different session, repo,
   or machine, so treat its claims as *claimed* until the working tree confirms
   them.

Ordered rule, no ambiguity: empty arg → session; arg is exactly `markdown`/`md`
→ session+file; otherwise → forensics. (If someone leads with `markdown` *and*
pastes a blob, the flag wins: treat it as session+file and ignore the rest, or
ask — don't silently half-do both.)

## Gather context (read-only commands only)

In a git repo, run these to see what actually changed. They are all safe — they
read, they never mutate:

- `git status --porcelain=v1` — staged / unstaged / untracked at a glance
- `git diff --stat` and `git diff` — unstaged changes (the actual edits)
- `git diff --staged --stat` and `git diff --staged` — staged changes
- `git log --oneline -15` — recent commits; spot ones the agent just made
- `git stash list` — stashed work that might be the "lost" thing the user fears

If `git diff` is enormous, don't dump it — summarize per file/group and offer
markdown mode for the full detail.

For session mode, your other primary source is **the conversation above**: what
the agent said it was doing, the tool calls it made, commands it ran, errors it
hit, corrections the user gave. Read it like evidence, not like your own memory —
look for what was *claimed* vs what the git state actually shows.

## Brevity discipline (this is the whole point)

The user ran `/wtf` to *escape* a wall of text — an agent already buried them in
diffs and a vague "done." A bloated report is the same failure in a new costume.
So the report earns every line:

- **State each finding exactly once.** The EUR currency flip, the stray scratch
  files, the skipped typecheck — each belongs in its single most relevant section.
  Everywhere else, point to it ("see What changed"), don't re-describe it. If you
  catch yourself explaining the same fact in What-broke *and* Suspicious-assumptions
  *and* the checklist, collapse it to one home.
- **Conditional sections are terse bullets, not paragraphs.** One line per item:
  what it is + why it matters. Save the prose for the verdict and Intent.
- **Length tracks chaos, not repo size.** Three real problems → a short report,
  even in a huge repo. Don't pad a clean-ish session to look thorough; don't let a
  genuinely messy one sprawl past what a tired human will actually read. If there
  are 20 findings, lead with the 3 that bite and group the rest.
- **The verdict + core-four stay tight regardless.** Those are the skim path; they
  must read fast even when the detail below is long.

A good report is one a stressed human reads top to bottom without skipping. If a
section doesn't change what they'll *do* next, cut it.

## Report structure

Lead with a three-line verdict so the user gets the gist before any detail:

```
WTF — quick verdict
  Tried:   <one line: what the agent was attempting>
  Changed: <N files, M insertions, K deletions — the headline>
  Watch:   <the single biggest thing to be suspicious of, or "looks clean">
```

Then the **core four** — always render these, even in a clean session:

### 1. Intent
What the agent was trying to do, in plain language. From the conversation
(session mode) or the paste (forensics). One short paragraph.

### 2. What changed
Files touched, grouped sensibly, each with a one-line *why* and its `+/-` stat.
Tie each change back to the intent where you can. If a file changed for no reason
you can find in the intent, say so — that's a signal, not a footnote.

### 3. What broke
Failing commands, tests, builds, type errors — quoted **exactly**, not
paraphrased. This is the one section where "Nothing broke that I can see" is a
fine and useful answer, because it's the reassurance the user came for. If you
can't tell whether something broke (no command was run), say that explicitly
rather than implying all-clear.

### 4. Safest next prompt
A ready-to-paste instruction the user can hand the agent (or run themselves) to
move forward safely — usually "verify before continuing": run the typecheck/
tests/build that should have run, or revert the risky piece first. Make it
concrete and specific to what you found.

Then the **conditional sections** — include each one *only when it has real
content*, and only when it adds something the core four didn't already say. Empty
"none" rows are noise; so is a section that just re-lists findings from above.
Terse bullets here, one line each. Skip any section whose content is fully
covered by a pointer back to the core four.

- **Commands & tests run** — what the agent actually executed, and the result.
- **Should have run but didn't** — checks the changes clearly warranted that
  never happened. Infer from what changed, then name the *real* command (read the
  repo's `package.json` scripts, `Makefile`, `CLAUDE.md`, CI config) instead of
  guessing a generic one. E.g. edited `*.ts/tsx` with no typecheck/build → name
  the actual `pnpm`/`npm` script; touched a migration with no review → flag it;
  changed `*.py` with no test run → name the test command. The point is to catch
  the verification gap the agent skipped.
- **Suspicious assumptions** — guesses the agent made and ran with: invented
  values, assumed file locations, "I'll assume you want X" moments, API shapes it
  never verified.
- **Risky or unrelated edits** — changes outside the stated intent: files touched
  that nobody asked about, scope creep, drive-by edits, things that look like
  collateral damage.
- **Rollback recipes** — copy-paste commands to undo specific pieces, so the user
  can surgically revert the risky bits without nuking the good work. Print them;
  never run them. Examples:
  - one file: `git restore --staged --worktree -- path/to/file`
  - a committed change: `git revert <sha>` (safe, makes a new commit) or
    `git checkout <sha>~1 -- path/to/file` to grab the prior version
  - recover a stash: `git stash show -p stash@{0}` then `git stash apply stash@{0}`
  Always pick the *least destructive* option that does the job, and say what each
  one does so the user chooses with eyes open.
- **Human review checklist** — only the spots where your read-only analysis
  genuinely can't decide and a human must. This is *not* a recap of the report —
  if an item is already covered above, leave it out. Often this section is empty
  or two lines; that's fine.

Watch the overlap between **Safest next prompt**, **Should have run**, and the
**checklist** — all three drift toward "run typecheck/test/build." Name the actual
checks once (in Should have run). Let the next prompt *reference* them ("run the
checks above, paste output") rather than re-listing the commands. Don't make the
user read the same three commands three times.

## Should-have-run inference

This is one of the most valuable parts — agents routinely edit code and declare
victory without verifying. Map what changed to what should have been checked,
using the *repo's actual* commands:

1. See which file types/areas changed.
2. Read `package.json` scripts / `Makefile` / `CLAUDE.md` / CI files to find the
   project's real verification commands.
3. If a warranted check appears nowhere in the session, flag it and quote the
   exact command the user should run.

Don't invent a generic `npm test` if the repo uses `pnpm test --filter=api`. The
specificity is the value.

## Writing the file (markdown mode only)

Only when the user invoked `/wtf markdown` (or `md`):

1. Ensure the output dir is git-ignored **before** writing, so the report never
   gets committed by accident:
   - If `.gitignore` exists and has no `.wtf/` line, append `.wtf/` to it.
   - If there's no `.gitignore`, create one containing `.wtf/`.
2. Write the full report (all sections, fully expanded — the file is the
   archive, so don't elide the big diff summary the way you might in the
   terminal) to:
   `.wtf/wtf-$(date +%Y-%m-%d-%H%M%S).md`
   Timestamped so runs never overwrite each other.
3. Still print the report to the terminal too — the file is in addition, not
   instead.
4. End by printing the written path so the user can open it.

## Edge cases

- **Not a git repo** → skip all git steps; build the report from the conversation
  or paste alone, and say plainly that there's no git context to verify against.
- **Clean tree, nothing in the session to dissect** → short report: nothing
  happened worth a post-mortem. Suggest the user paste a transcript/diff if the
  thing they're confused about lives in another session or tool.
- **Forensics paste that doesn't match the working tree** → analyze the paste on
  its own terms and label its file changes as *claimed* / unverified, since you
  can't confirm them against this repo.
- **Huge diff** → summarize by file and group; never wall-of-text the raw diff.
  Point to markdown mode for the complete record.

## Hard safety rule

Never run a mutating command as part of this skill: no `git restore`,
`checkout --`, `reset`, `clean`, `stash drop/pop`, `commit`, `push`, no file
deletes or rewrites of the user's work (the `.wtf/` report and a `.gitignore`
append are the only writes you make). The user is confused and possibly about to
lose work — your value is a clear map and exact recipes, not taking the wheel.
Everything destructive is theirs to run after they've read what you found.
