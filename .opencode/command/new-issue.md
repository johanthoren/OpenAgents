---
description: Create well-structured GitHub issues with duplicate checking
---

# Create GitHub Issue

Create a new GitHub issue with duplicate detection and optional scratchpad.

## Input Parsing

Parse `$ARGUMENTS` for issue number or description:
- `42` or `#42` → View existing issue (redirect to `/work-on-issue 42`)
- Text description → Create new issue with this as starting point
- Empty → Prompt for description

**$ARGUMENTS:** $ARGUMENTS

## Workflow

### Step 1: Repository Check

```bash
git rev-parse --git-dir 2>/dev/null || echo "NOT_A_REPO"
```

If not a repo, exit: "Not in a git repository. Run `git init` first."

```bash
git remote get-url origin 2>/dev/null || echo "NO_REMOTE"
```

If no remote, exit: "No GitHub remote. Add one with `git remote add origin <url>`"

### Step 2: Get Description

If $ARGUMENTS is empty, ask:
```
What issue would you like to create? (brief description)
>
```

### Step 3: Check for Duplicates

Search for similar issues:
```bash
gh issue list --search "[keywords from description]" --state all --limit 5 --json number,title,state
```

If matches found, present:
```
Potentially related issues:
- #42: [title] (open)
- #38: [title] (closed)

Continue creating new issue? (y/n/view #)
>
```

If user says `n`, exit. If `view #`, show that issue and re-prompt.

### Step 4: Gather Details

```
Issue type? (bug/feature/refactor/docs) [feature]
>

Brief title (imperative mood, e.g., "Add user authentication"):
>

Success criteria (how do we know it's done?):
>

Priority? (low/medium/high) [medium]
>
```

### Step 5: Create Issue

Build issue body:
```markdown
## Problem
[From user description]

## Success Criteria
[From user input]

## Priority
[From user input]
```

Create:
```bash
gh issue create --title "[title]" --body "[body]" --label "[type]"
```

Capture the issue number from output.

### Step 6: Create Scratchpad (Optional)

```
Create a scratchpad for planning? (y/n) [y]
>
```

If yes:
```bash
mkdir -p scratchpads
```

Write to `scratchpads/issue-[number]-[slug].md`:
```markdown
# Issue #[number]: [Title]

Link: [URL]
Created: [date]

## Problem
[Description]

## Success Criteria
[Criteria]

## Implementation Notes
<!-- Add notes as you work -->
```

### Step 7: Summary

```
✅ Created issue #[number]: [title]
   URL: [url]
   Labels: [labels]
   Scratchpad: scratchpads/issue-[number]-[slug].md

Next: Run `/work-on-issue [number]` to start working on it.
```

## Error Handling

- `gh` not installed → "Install GitHub CLI: https://cli.github.com"
- Not authenticated → "Run `gh auth login` first"
- Creation fails → Save draft to `failed-issue-[timestamp].md`
