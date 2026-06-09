---
name: tidy
description: Clean up and organize a project's file structure — group loose files into logical folders (images/, scripts/, styles/, etc.), remove empty directories, and delete genuinely unnecessary files. Use when the user asks to "clean up the project", "organize the files", "tidy the directory", "sort these into folders", "remove empty folders", or "get rid of junk files". Always understands the project and updates references BEFORE moving anything, never deletes load-bearing files, and gets approval before changing the file tree.
---

# Project Tidy — Organize, Don't Break

Make a messy project clean and easy to manage **without breaking it**. Moving or
deleting files can silently break a build, a page, or an import. So the rule is:
**understand first, plan, get approval, then execute — and update every reference.**

This skill can move and delete files. Treat that power carefully. When unsure
whether something is safe to move or delete, **leave it and ask.**

## Step 1 — Understand the project

Before touching anything:

- Map the current tree (`git ls-files`, directory listing). Identify the project
  type (static site, Node app, Python package, etc.) and its **entry points**
  (`index.html`, `main.py`, `src/index.js`, etc.).
- Read build/config files (`package.json`, bundler config, framework config) —
  they often hardcode paths and dictate where files must live.
- Understand **how files reference each other**: HTML `src`/`href`, CSS `url()` and
  `@import`, JS/TS `import`/`require`, asset paths in code, config path strings.
- Check what's tracked vs untracked (`git status`) so you don't reorganize generated
  output (`dist/`, `node_modules/`) — those belong in `.gitignore`, not in folders.

## Step 2 — Plan the reorganization (don't execute yet)

Propose a clean structure based on what you found. Typical pattern for a flat
web project with loose files:

```
before/                      after/
  index.html                   index.html          ← entry stays at root
  about.html                   about.html
  style.css                    styles/style.css
  main.js                      scripts/main.js
  logo.png                     images/logo.png
  photo1.jpg ... photo20.jpg   images/photo1.jpg ...
```

Guidelines:
- Group by role: `styles/`, `scripts/` (or `js/`), `images/` (or `assets/`),
  `fonts/`, `docs/`. Follow any convention the project already uses.
- **Keep entry points and required-root files where they belong** — `index.html`,
  `package.json`, `README`, `LICENSE`, `.gitignore`, CI configs, framework configs.
  Don't bury them in subfolders.
- Don't invent deep nesting for a tiny project. Clean ≠ over-engineered.

Show the user the proposed before→after tree and the list of deletions **before
doing it.** Get approval. This is a file-tree change — confirm first.

## Step 3 — Move files AND fix every reference

When you move a file, you MUST update everything that points to it, or you break the
project. For each moved file:

- Find all references (grep the codebase for the old path/filename).
- Update them to the new path: HTML `<img src>`/`<link href>`/`<script src>`,
  CSS `url(...)`/`@import`, JS/TS imports, and any config/string paths.
- Move with `git mv` when tracked (preserves history) rather than plain `mv`.

After moving, verify nothing dangles: re-grep for the old paths; if the project
builds or has tests, run them to confirm it still works.

## Step 4 — Remove empty directories

Empty directories are clutter — remove them. **But** first check they're truly
disposable:

- A dir kept intentionally usually has a `.gitkeep`/`.keep` marker — that signals
  "keep this empty dir." Don't delete those (or confirm with the user).
- Some tooling/frameworks expect certain dirs to exist even when empty. If a dir is
  referenced in config or is a known framework path, leave it.

## Step 5 — Delete unnecessary files (carefully)

The user wants junk gone — but **an empty or tiny file is not automatically junk.**
Some tools create load-bearing files on purpose. Before deleting ANY file, even an
empty one:

1. **Confirm it's not referenced** anywhere (grep imports/paths/config).
2. **Confirm it's not a known "load-bearing even when empty" file.** Never delete
   these just because they look empty:
   - `__init__.py` (Python — makes a package importable)
   - `.gitkeep` / `.keep` (intentionally preserves a dir)
   - `py.typed` (marks a typed Python package)
   - `index.js` / `index.ts` / `mod.rs` / `__init__` barrels (module entry points)
   - `.gitignore`, `.npmignore`, `.dockerignore`, `.editorconfig`, `.nvmrc`
   - `package.json`, lockfiles, `go.mod`, `Cargo.toml`, manifests
   - `.env.example`, config stubs, `robots.txt`, `favicon.ico`, `.htaccess`
   - CI/workflow files, `Dockerfile`, `Makefile`
3. **Only then** treat it as a deletion candidate. Genuinely safe candidates:
   editor/OS cruft (`.DS_Store`, `Thumbs.db`, `*.swp`), stray temp files
   (`*.tmp`, `*.bak`, `*~`), duplicate/leftover files the user confirms.

When in doubt, **list it as "suspected unnecessary — confirm?" rather than deleting.**
It is always better to leave a harmless file than to delete a load-bearing one.

## Step 6 — Report

Summarize what changed: files moved (old → new), references updated, empty dirs
removed, files deleted, and anything you deliberately left alone with the reason.
If a build/test was available, state that it still passes.

## Rules

- **Plan and get approval before changing the tree.** No surprise moves/deletes.
- **Update references whenever you move a file** — a move without fixing paths is a
  bug, not a cleanup.
- **Never delete a load-bearing file**, even if empty. When unsure, keep and ask.
- Use `git mv` / `git rm` for tracked files so history and staging stay correct.
- Don't reorganize generated/dependency dirs — gitignore them instead.
- Prefer the project's existing conventions over imposing a new structure.
