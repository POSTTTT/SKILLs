---
name: qna
description: Answer questions, explain code, and investigate the codebase WITHOUT making any changes. Use when the user wants discussion, analysis, a plan, or a recommendation only — e.g. "just answer, don't change anything", "explain how X works", "what would you change?", "don't edit yet", "Q&A only", or any question where the user has not clearly asked you to modify files.
---

# Q&A Mode — Answer, Don't Change

You are in **read-only Q&A mode**. The user wants understanding, analysis, or a
recommendation — **not** modifications. Treat this as an investigation-and-explain
task, never an implementation task.

## Hard rules

While this skill is active, you MUST NOT:

- Edit, write, create, move, or delete any file (no `Edit`, `Write`, `NotebookEdit`).
- Run any command that changes state — no `git commit`/`add`/`push`/`checkout`,
  no installs, no migrations, no codegen, no `mkdir`/`rm`/`mv`, no formatters or
  linters run in `--fix`/`--write` mode.
- Stage, branch, or otherwise alter version control.

You MAY freely:

- Read files, search (`Grep`/`Glob`), and inspect the project.
- Run **read-only** shell commands (`git status`, `git log`, `git diff`,
  `ls`, `cat`-equivalents, test/build commands **only to observe**, never to fix).
- Reason out loud, explain, compare options, and draft what code *would* look like
  inside your reply as a fenced code block — clearly labeled as a proposal, not applied.

## How to answer

1. **Investigate first.** Read the relevant code before answering. Cite specifics
   as `path/to/file.ext:line` so the user can jump straight there.
2. **Answer the actual question.** Be direct. Lead with the conclusion, then the
   supporting detail.
3. **If a change would be the natural next step, describe it — don't do it.**
   Show the proposed diff or snippet in your message and explain the trade-offs.
4. **End with an explicit handoff.** Offer to make the change, e.g.:
   > "Want me to apply this? Say the word and I'll make the edit."
   Do not proceed until the user clearly approves.

## When the user approves changes

If the user explicitly says to go ahead (e.g. "yes, make the change", "do it",
"apply it"), this skill's restriction is lifted **for that change** — implement it
normally. When in doubt about whether approval was given, ask.

## Note on Plan Mode

If the user wants a hard, harness-enforced guarantee that nothing is edited,
mention that they can press **Shift+Tab** to enter Claude Code's built-in Plan
Mode, which blocks edit tools entirely. This skill is the lighter-weight,
ask-in-natural-language version of the same intent.
