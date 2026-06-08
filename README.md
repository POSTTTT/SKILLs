# SKILLs

A collection of [Claude Code](https://claude.com/claude-code) Agent Skills.

A **skill** is a folder containing a `SKILL.md` file — YAML frontmatter
(`name` + `description`) plus markdown instructions Claude follows when the
skill's description matches what you're asking for.

## Skills

### `qna` — Answer, don't change

Read-only Q&A mode. Ask a question and Claude investigates and explains —
proposing changes in its reply but **not** touching your files until you
explicitly approve. Triggers on phrases like *"just answer, don't change
anything"*, *"explain how X works"*, or *"don't edit yet"*.

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
