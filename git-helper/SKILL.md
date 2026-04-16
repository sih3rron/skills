---
name: git-helper
description: >
  Shorthand git workflow skill for common operations: staging, committing (with
  auto-generated commit messages from git diff --staged), pushing, pulling, and
  creating GitHub PRs via the gh CLI. Use this skill whenever the user asks to
  commit, push, pull, stage files, open a PR, or says anything like "commit my
  changes", "push this up", "make a PR", "what's staged", or "ship it". Also
  trigger when the user describes changes they've made and implies they want to
  save or share them — even if they don't use explicit git vocabulary.
---

# Git Helper

Handles common git operations via natural language. Claude runs the commands
using `bash_tool` and narrates what it's doing.

---

## Core Operations

### 1. Commit (with auto-generated message)

When the user asks to commit without providing a message:

1. Run `git diff --staged` to read what's staged
2. If nothing is staged, run `git status` and offer to stage relevant files
3. Generate a concise conventional commit message from the diff:
   - Format: `<type>(<scope>): <short summary>` (≤72 chars)
   - Types: `feat`, `fix`, `chore`, `refactor`, `docs`, `style`, `test`
   - Scope: infer from file paths (e.g. `auth`, `ui`, `api`)
   - If the diff is too large or mixed, ask the user to confirm the message before committing
4. Run: `git commit -m "<generated message>"`
5. Show the user the message used

If the user provides their own message, skip steps 1–3 and commit directly.

**Example triggers:** "commit my changes", "commit this", "ship a commit", "save my work"

---

### 2. Stage files

When the user wants to stage:

- "stage everything" → `git add -A`
- "stage <file or glob>" → `git add <path>`
- "stage my changes" (ambiguous) → run `git status`, show unstaged files, ask which to stage

---

### 3. Push

- `git push` by default
- If the branch has no upstream: `git push --set-upstream origin <current-branch>`
- Show the remote URL and branch being pushed to

**Example triggers:** "push", "push this up", "push to origin", "send it"

---

### 4. Pull

- `git pull` by default
- If there are local uncommitted changes, warn the user and suggest stashing first

**Example triggers:** "pull", "pull latest", "get the latest", "sync with main"

---

### 5. Create a PR (GitHub CLI)

Requires `gh` to be installed and authenticated.

1. Run `git log origin/<base>..HEAD --oneline` to summarise commits vs base branch
2. Infer a PR title from the branch name or recent commits
3. Generate a short PR body summarising what changed (from the commit log)
4. Run: `gh pr create --title "<title>" --body "<body>" --base <base>`
   - Default base branch: `main` (fall back to `master` if `main` doesn't exist)
   - If the user specifies a base branch, use that
5. Output the PR URL

**Example triggers:** "open a PR", "create a pull request", "make a PR", "raise a PR"

---

## Guardrails

- **Never force-push** unless the user explicitly says "force push". If they do, confirm before running.
- **Never commit to `main` or `master` directly** — warn the user and suggest creating a branch.
- **Confirm before pushing** if the current branch is a protected branch name (`main`, `master`, `release/*`, `production`).
- If `gh` is not installed or not authenticated, tell the user and suggest running `gh auth login`.

---

## Status helpers

Run these proactively if context is unclear:

- `git status` — what's staged, unstaged, untracked
- `git branch --show-current` — current branch name
- `git log --oneline -5` — recent commits for context

---

## Output style

- Keep it brief. Show the command run and its output.
- If a commit message was auto-generated, show it and let the user correct it before confirming.
- Use inline code formatting for commands and file paths.
