---
name: gh-pr-create
description: >-
  Use when opening a pull request, submitting code for review, or when the user
  says "create PR," "open PR," or "/gh-pr-create." Generates conventional-commit
  title and structured body from branch commits.
---

# GH PR Create

Generate and submit a GitHub pull request from the current feature branch.

**Core principle:** Synthesize, don't dump. A PR tells reviewers *why*, not just *what*.

**Announce at start:** "I'm using the gh-pr-create skill to create a pull request."

## The Iron Laws

```
1. NO PR WITHOUT USER PREVIEW
2. NO RAW COMMIT DUMPS — ALWAYS SYNTHESIZE
```

Create a PR the user hasn't approved? Violation. Copy-paste commit messages as the body? Violation.

## The Process

### Step 1: Validate Prerequisites

Run checks in order. STOP at first failure.

```bash
git rev-parse --is-inside-work-tree          # Must be a git repo
gh auth status                               # Must be authenticated
BASE=$(gh repo view --json defaultBranchRef --jq '.defaultBranchRef.name')
HEAD=$(git branch --show-current)            # Must differ from $BASE
git log $BASE..HEAD --oneline                # Must have commits
gh pr view --json url 2>&1                   # Must NOT have existing PR
```

<HARD-GATE>GIT REPO — must be inside a git repository</HARD-GATE>
<HARD-GATE>GH AUTH — gh auth status must succeed</HARD-GATE>
<HARD-GATE>NOT DEFAULT BRANCH — current branch must differ from default</HARD-GATE>
<HARD-GATE>COMMITS AHEAD — must have commits ahead of base</HARD-GATE>

**Dirty working tree:** Run `git status --porcelain`. If non-empty, warn: "You have uncommitted changes that won't be included in this PR." Proceed anyway.

### Step 2: Push Branch

```bash
git rev-parse --abbrev-ref @{upstream} 2>&1  # Check upstream
git push -u origin $HEAD                     # Push if needed
```

Skip if upstream already set and up to date.

### Step 3: Gather Context & Generate Content

```bash
git log $BASE..HEAD --pretty=format:'%h %s' --reverse  # Commits
git diff --stat $BASE..HEAD                              # Diff stats
git diff --name-only $BASE..HEAD                         # Changed files
```

**Detect PR template** (check in order):
1. `gh repo view --json pullRequestTemplates` — use first if multiple
2. Local filesystem: `.github/PULL_REQUEST_TEMPLATE.md`, `.github/pull_request_template.md`, `.github/PULL_REQUEST_TEMPLATE/*.md` (first in dir), `docs/pull_request_template.md`, `PULL_REQUEST_TEMPLATE.md`
3. Fallback: [references/pr-template.md](references/pr-template.md)

**Generate title** — conventional-commit format: `type(scope): subject`

**Fill the body:**
- **Single commit:** Use the commit's full message (subject + body) to fill the template. No synthesis needed.
- **Multiple commits:** Synthesize a summary from all commits. Group changes by logical area.
- **Low-quality commits** (e.g., "fix", "wip", "stuff"): Ask the user "What did you change and why?" Combine their answer with diff stats to generate the body.

**Linked issues:** Ask about linked issues. Verify with `gh issue view <N>`. Append `Closes #X`.

**Irrelevant template sections:** Write "N/A" — don't delete them.

### Step 4: Preview

Show in this exact format:

```
══════════════════════════════════════
PR PREVIEW
══════════════════════════════════════

Title: type(scope): subject

Base: main ← Head: feature-branch

Body:
──────────────────────────────────────
[full body content]
──────────────────────────────────────

Closes: #123, #456 (or "None")

══════════════════════════════════════
```

Ask: **"Create this PR? (yes / edit / cancel)"**

<HARD-GATE>USER APPROVAL — preview must be approved before creation</HARD-GATE>

### Step 5: Create

```bash
echo "$BODY" | gh pr create --base "$BASE" --title "$TITLE" --body-file -
```

Print PR URL. If issues linked, note they'll auto-close on merge.

## Conventional Commit Title

Format: `type(scope): subject`

| Type | Use for |
|------|---------|
| feat | New feature |
| fix | Bug fix |
| docs | Documentation only |
| style | Formatting, no logic change |
| refactor | Neither fix nor feature |
| test | Adding or updating tests |
| chore | Build, tooling, deps |
| ci | CI/CD changes |
| perf | Performance improvement |

- Subject: imperative, lowercase, no period, under 50 chars
- Scope: optional, derived from primary area of change
- Single commit: use its message if already conventional; rewrite if not
- Multiple commits: synthesize dominant type + overall subject

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Creating from default branch | Hard gate catches this |
| Not pushing first | Step 2 handles auto-push |
| Listing commits as body | Synthesize a summary (Iron Law 2) |
| Ignoring repo template | Always check before falling back |
| Hardcoding `main` as base | Auto-detect via `gh repo view` |
| Skipping issue question | Always ask about linked issues |
| Creating without preview | Hard gate + Iron Law 1 |

## Red Flags — STOP

| Thought | Reality |
|---------|---------|
| "I'll just create it, user can edit later" | Iron Law 1: preview first |
| "The base branch is probably main" | Detect it, never assume |
| "I'll list the commits in the body" | Iron Law 2: synthesize |
| "Commits are clear enough, no need to ask" | Low-quality commits need user input |
| "This template section doesn't apply, I'll delete it" | Write "N/A," don't delete |
| "No issues to link, I'll skip asking" | Always ask |
