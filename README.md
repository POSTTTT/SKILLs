# SKILLs

A collection of [Claude Code](https://claude.com/claude-code) Agent Skills.

A **skill** is a folder containing a `SKILL.md` file — YAML frontmatter
(`name` + `description`) plus markdown instructions Claude follows when the
skill's description matches what you're asking for.

## Skills

Skills are grouped by whether they operate on a **codebase** (built for Claude
Code, where they have access to your local files and git) or are **general-purpose**
(work on documents/content and are portable to the Claude apps too).

## Code skills

These read and act on a local project — best used in **Claude Code**.

### `qna` — Answer, don't change

Read-only Q&A mode. Ask a question and Claude investigates and explains —
proposing changes in its reply but **not** touching your files until you
explicitly approve. Triggers on phrases like *"just answer, don't change
anything"*, *"explain how X works"*, or *"don't edit yet"*.

### `security-audit` — OWASP Top 10 (2025) + hardening review

Audits a codebase for security weaknesses against the
[OWASP Top 10 (2025)](https://owasp.org/Top10/2025/) plus seven hardening areas:
secrets management, data encryption, input validation/injection, auth,
dependency/supply-chain, error handling/logging, and configuration/transport
hardening. On invocation it asks **which area(s) to investigate (or all)**, then
reports findings with severity, `file:line`, and remediation — read-only, no code
changes until you ask. Triggers on *"do a security review"*, *"find
vulnerabilities"*, *"check for hardcoded secrets / SQL injection"*, etc.

### `gitignore` — Understand the repo, then write `.gitignore`

Analyzes the codebase to detect its languages, frameworks, and tooling, then
creates or updates `.gitignore` with the right entries — grouped and commented,
merged with any existing rules (no duplicates, never clobbers lockfiles or
`.example` templates). Also flags files that are *already tracked* and need
`git rm --cached`. Triggers on *"fix/generate the .gitignore"*, *"what should I
gitignore"*, *"stop tracking node_modules/.env"*, etc.

### `tidy` — Organize the project, don't break it

Cleans up a messy file tree: groups loose files into logical folders
(`images/`, `scripts/`, `styles/`…), removes empty directories, and deletes
genuine junk. It understands the project first, **updates every reference when it
moves a file** (so `<img src>`/imports don't break), **never deletes load-bearing
files** even when empty (`__init__.py`, `.gitkeep`, manifests…), and shows a
before→after plan for approval before changing anything. Triggers on *"clean up
the project"*, *"organize the files"*, *"remove empty folders"*, etc.

### `pre-push` — Is it safe to push right now?

A pre-flight safety gate. Inspects the **actual git state** — staged diff, tracked
files, source, history — for secrets/API keys, `.env`/`.venv` being tracked, PII,
leftover conflict/debug markers, and `.gitignore` gaps, then gives a clear
✅ ready / ⛔ blockers verdict. Read-only; surfaces `git rm --cached` and reminds
you to *rotate* any secret already in history, and defers to `gitignore` and
`security-audit` for fixes. Triggers on *"is this ready to push?"*, *"can I commit
this?"*, *"did I leave any secrets in?"*.

### `optimize` — Performance report (measure, don't guess)

Analyzes the codebase and writes a performance report structured around five
principles: do less work (algorithmic complexity), respect the memory hierarchy
(data locality), measure first (profile + Amdahl's Law), exploit parallelism, and
cut overhead. Findings are ranked by **impact** with `file:line`, expected
speed/memory effect, effort, risk, and a measured-vs-suspected confidence tag — and
it flags premature/cold-path tweaks as not worth it. Reports first; rewrites hot
code only when you ask. Triggers on *"optimize the program"*, *"make it faster"*,
*"find bottlenecks"*, *"why is this slow?"*.

## Non-code skills

General-purpose skills that work on documents/content rather than a codebase.
These are also portable to the **Claude apps** (web/desktop) — upload the skill
folder as a ZIP via *Customize → Skills*.

### `pdf-summary` — Full, page-aware PDF summaries

Analyzes a PDF in its entirety and produces a structured, chapter-by-chapter
breakdown with **minimal compression**. Reads every page (in batches for long
docs), **cites where each point comes from** (page/section), explains diagrams and
figures and how they relate to the content, and keeps any instructions/procedures
**complete** rather than summarized. Triggers on *"summarize this PDF"*, *"analyze
this document"*, *"break down this PDF"*, or pointing at a `.pdf`.

## Install

### Quick (one line)

Using [Vercel Labs' `skills` CLI](https://github.com/vercel-labs/skills):

```bash
npx skills add POSTTTT/SKILLs
```

Then **restart Claude Code** — skills are loaded at startup, not dynamically.

### Install only the skills you want

Each skill is an independent folder, so you can install just the one you need:

```bash
npx skills add POSTTTT/SKILLs --list           # see what's available
npx skills add POSTTTT/SKILLs --skill qna      # install just one
```

Or run `npx skills add POSTTTT/SKILLs` with no flag and pick from the
interactive list.

### Manual (no extra tooling)

Copy a skill into your personal skills folder (available in every project on your machine):

```bash
# macOS / Linux
cp -r .claude/skills/qna ~/.claude/skills/

# Windows (PowerShell)
Copy-Item -Recurse .claude\skills\qna $HOME\.claude\skills\
```

### Project-only

Skills in `.claude/skills/` are available automatically when Claude Code runs
inside this repo — no install needed.

Restart Claude Code (or start a new session) and the skill is live.
