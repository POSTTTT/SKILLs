---
name: gitignore
description: Analyze the codebase to understand its languages, frameworks, and tooling, then create or update the project's .gitignore with the right entries. Use when the user asks to "fix/update/generate the .gitignore", "what should I gitignore", "add ignore rules", "stop tracking node_modules/.env/build artifacts", or wants secrets and generated files kept out of version control. Understands the repo first, avoids duplicates, and flags already-tracked files that ignoring alone won't remove.
---

# .gitignore Builder

Understand the codebase **first**, then write a correct, well-organized
`.gitignore`. Do not paste a generic template blindly — base every entry on
something actually present in this repo.

## Step 1 — Understand the codebase

Inspect the repo to learn what it's made of. Look for:

- **Languages & package managers:** `package.json` (Node), `requirements.txt` /
  `pyproject.toml` / `Pipfile` (Python), `go.mod` (Go), `Cargo.toml` (Rust),
  `pom.xml` / `build.gradle` (Java/Kotlin), `Gemfile` (Ruby), `composer.json` (PHP),
  `*.csproj` / `*.sln` (.NET), etc.
- **Frameworks & build tools:** React/Next/Vue/Vite, Django/Flask, Rails, Spring,
  webpack/rollup, etc. — each has signature output dirs.
- **Generated / dependency dirs already on disk:** `node_modules/`, `dist/`,
  `build/`, `.next/`, `target/`, `__pycache__/`, `venv/`, `.venv/`, `vendor/`,
  coverage output, compiled binaries.
- **Secrets & local config:** `.env`, `.env.*`, `*.pem`, `*.key`, credential files,
  local DB files (`*.sqlite`, `*.db`).
- **Editor / OS noise:** `.vscode/`, `.idea/`, `.DS_Store`, `Thumbs.db`, `*.swp`.
- **Logs / caches / temp:** `*.log`, `.cache/`, `tmp/`, `coverage/`.

Also:
- **Read the existing `.gitignore`** (if any) so you don't add duplicates and you
  respect its structure/comments.
- Run `git status --porcelain` and `git ls-files` to see what's actually tracked vs
  untracked — this reveals junk that's currently committed (e.g. a checked-in
  `node_modules/` or `.env`).

## Step 2 — Decide what to ignore

Build the list from evidence, not assumptions:

- Include rules only for stacks/tools you actually detected.
- Group entries under clear `# comment` headers (e.g. `# Node`, `# Python`,
  `# Secrets`, `# Editor`, `# OS`).
- Prefer canonical patterns (`node_modules/`, not `node_modules/*`).
- **Never ignore things that should be tracked:** lockfiles (`package-lock.json`,
  `poetry.lock`, `Cargo.lock`, `go.sum`), `.env.example` templates, source config.
  When ignoring env files, keep the example: `.env` + `.env.*` but `!.env.example`.

## Step 3 — Write the .gitignore

- If none exists, create one. If one exists, **append/merge** — keep the user's
  existing entries and comments, only add what's missing. Never silently delete
  their rules.
- Keep it organized and commented so it stays maintainable.
- Show the user the proposed additions (a short diff/summary) before or alongside
  writing, so they know what changed.

## Step 4 — Flag already-tracked files (important)

Adding a pattern to `.gitignore` does **not** untrack files Git already tracks. If
Step 1 found tracked files that now match an ignore rule (e.g. a committed `.env` or
`node_modules/`), tell the user and offer the fix:

```bash
git rm -r --cached <path>     # stop tracking, keep the local file
# then commit the removal
```

If a **secret** (like a committed `.env` or key) is already in history, warn that
removing it from tracking does NOT scrub git history — recommend rotating the secret
and, if needed, history-rewriting tools (`git filter-repo`, BFG).

## Rules

- Evidence-based: every entry should correspond to something in this repo (or a
  near-certain artifact of a detected tool). Skip stacks that aren't present.
- Don't clobber an existing `.gitignore` — merge, don't replace.
- Don't ignore lockfiles or `.example` templates.
- Surface (don't auto-run) `git rm --cached` for already-tracked matches; let the
  user confirm before changing what's tracked.
