---
name: pre-push
description: Pre-flight safety gate that checks whether the project is actually safe to commit/push to GitHub (or any remote) RIGHT NOW. Use when the user asks "is this ready to push?", "can I commit this?", "check before I push", "did I leave any secrets in", "ready for GitHub?". Inspects the real git state — staged diff, tracked files, source, and history — for secrets/API keys, .env/.venv being tracked, PII, leftover debug/conflict markers, and .gitignore gaps. Read-only; reports a clear ready/blockers verdict and defers to the gitignore and security-audit skills for fixes.
---

# Pre-Push Readiness Gate

Answer one question: **"If I push right now, will I regret it?"** This is a fast,
focused pre-flight check of what is *actually about to be published* — not a deep
code review. Run read-only, then give a clear verdict.

Key principle: **a file being in `.gitignore` does NOT mean it's safe.** `.gitignore`
only affects *untracked* files. A secret that was committed before being ignored, a
key hardcoded in source, or a secret in history will still push. This gate checks
**reality** (git state), not just the rules.

## What this skill is (and isn't)

- It **is** a gate: a quick checklist run right before commit/push.
- It is **not** the `gitignore` skill (which writes ignore rules) or `security-audit`
  (deep OWASP review). When this gate finds a rules gap or wants depth, it **defers**:
  "Your `.gitignore` is missing X — want me to run the `gitignore` skill?" Skills
  can't call each other programmatically, so recommend and let the user trigger them.

## Step 1 — Read the real git state

- `git status --porcelain` — what's staged / modified / untracked.
- `git diff --cached` — exactly what the next commit will contain.
- `git ls-files` — what's already tracked (the stuff `.gitignore` can't save you from).
- Current branch (`git branch --show-current`) and whether commits are ahead of remote.

## Step 2 — Run the checklist

### A. Secrets about to ship  (highest priority)
Scan the staged diff, tracked files, AND source contents for:
- API keys / tokens: `api[_-]?key`, `secret`, `token`, `Bearer `, `AKIA[0-9A-Z]{16}`
  (AWS), Google/Stripe/GitHub token shapes, high-entropy strings.
- Private keys: `-----BEGIN (RSA |EC |OPENSSH )?PRIVATE KEY-----`.
- Hardcoded passwords / connection strings (`password=`, `mongodb://user:pass@`).
- Secret files being **tracked**: `.env`, `.env.*` (except `.env.example`), `*.pem`,
  `*.key`, credential JSON, `.npmrc`/`.pypirc` with tokens.
- **Check both:** is the secret file *tracked* (`git ls-files`)? Is a secret *hardcoded
  in committed source*? Either is a blocker even if `.gitignore` lists the file.

### B. Environment / dependency dirs being tracked
- `.venv/` `venv/` `env/` (Python), `node_modules/`, `__pycache__/`, build output
  (`dist/`, `build/`, `.next/`, `target/`). If tracked, that's bloat/leakage — flag it.

### C. Personal / sensitive information (PII)
- Real emails, full names, phone numbers, addresses, real customer data, internal
  hostnames/IPs in committed content. Distinguish placeholders from real data.

### D. Repo hygiene
- Leftover merge-conflict markers: `<<<<<<<`, `=======`, `>>>>>>>`.
- Debug leftovers: `console.log` dumps, `debugger`, `print()` debugging, `TODO/FIXME`
  that block release, commented-out secrets.
- Large/binary files that shouldn't be in git (consider Git LFS).
- Empty or nonsense commit, or committing on the wrong branch (e.g. straight to `main`).

### E. .gitignore sanity
- Are `.env`, `.venv`, `node_modules`, build dirs ignored **and not already tracked**?
- If rules are missing → defer to the `gitignore` skill.

## Step 3 — Verdict

Give a clear, scannable result:

```
Pre-push check — <branch> — <date>

⛔ BLOCKERS (do not push)
  - [SECRET] AWS key hardcoded in src/config.js:12
  - [TRACKED] .env is tracked (git ls-files) — will publish DB password

⚠️  WARNINGS (review)
  - .venv/ is tracked (412 files) — bloat
  - Leftover console.log at src/app.js:88

✅ OK
  - No conflict markers, .gitignore covers build output

Verdict: NOT READY — 2 blockers.
```

Always end with the safe next actions, e.g.:
- Untrack a committed file (keep it locally):
  ```bash
  git rm --cached .env && echo ".env" >> .gitignore
  ```
- **If a secret already reached a commit/history:** removing it from tracking does NOT
  scrub history. Recommend **rotating the secret immediately**, and history rewrite
  (`git filter-repo` / BFG) if it must be purged.
- Offer to run `gitignore` (fix rules) or `security-audit` (deep review) as follow-ups.

## Rules

- **Read-only.** Report the verdict; don't commit, push, or edit. Fix only what the
  user approves, and surface `git rm --cached` rather than running it silently.
- **Blocker vs warning:** secrets/keys/PII about to publish = BLOCKER. Bloat, debug
  leftovers, style = WARNING. Be explicit which is which.
- **Don't echo full secret values** — show the location and a masked snippet; the goal
  is to flag, not to reprint the secret.
- **Rotate, don't just delete:** any secret that already reached a commit must be
  treated as compromised — say so.
- **Defer, don't duplicate:** lean on `gitignore` and `security-audit` for fixes/depth.
