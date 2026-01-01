---
description: Work on GitHub issues with guided workflow from selection to completion
---

# Work on GitHub Issue

Orchestrate issue resolution through subagent delegation: planning → TDD → implementation → validation → completion.

## Phase 1: Select & Load Issue

**Parse $ARGUMENTS for issue number** (formats: `42`, `#42`, `issue-42`).

If no issue number:
```bash
gh issue list --state open --limit 10 --json number,title,labels,updatedAt
```
Present list, prompt for selection or search query.

**Fetch issue context:**
```bash
gh issue view [number] --json number,title,body,state,labels,assignees,comments
```

Check for existing work:
```bash
find scratchpads -type f -name "issue-[number]-*.md" 2>/dev/null
gh pr list --search "[number]" --state all --json number,title,state
```

Present summary:
```
Issue #[number]: [Title]
Labels: [labels] | Assignees: [assignees or "unassigned"]

[Issue body - truncated if long]

Context: Scratchpad [exists/none] | Related PRs: [count]
```

## Phase 2: Plan

**Quick clarification:**
```
1. Branch or direct to master? (branch recommended for 4+ files)
2. Any constraints not in the issue? (or 'none')
```

**Assess complexity:**

| Criteria | Direct (master) | Branch + PR |
|----------|-----------------|-------------|
| Files affected | 1-3 | 4+ |
| Estimated time | <30 min | >30 min |
| Risk level | Low (docs, config) | Medium+ (logic, API) |
| Review needed | No | Yes |

**If complex (4+ files OR multi-component):**
```
DELEGATE to subagents/core/task-manager
PASS:
  - Issue body and acceptance criteria
  - Constraints from user
  - Branch decision
RECEIVE:
  - Subtask plan with sequences
  - Files affected list
  - Dependencies mapped
```

**If simple:** Create brief plan directly:
```
Implementation Plan:
- Tests: [what to test]
- Files: [what to modify]
- Commits: [estimated count]

Proceed? (y/n/suggest changes)
```

**Create branch if needed:**
```bash
git checkout -b [fix|feature|refactor]/issue-[number]-[brief-description]
```

## Phase 3: TDD Tests

**Always delegate to tester subagent:**
```
DELEGATE to subagents/code/tester
PASS:
  - Objective: [from issue body]
  - Acceptance criteria: [extracted from issue]
  - Files to test: [from plan]
RECEIVE:
  - Test files created
  - All tests failing (verified)
  - Test summary
```

**After delegation returns:**
- Verify tests fail as expected
- Commit failing tests:
  ```
  test: add failing tests for issue #[number]
  
  Tests define expected behavior. Currently failing - implementation to follow.
  ```

**If no test framework:** Ask user to choose:
1. Set up testing first (recommended)
2. Proceed without tests (document why)
3. Add tests after implementation

## Phase 4: Implement

### 4.1 Code Implementation

**Delegate to coder-agent:**
```
DELEGATE to subagents/code/coder-agent
PASS:
  - Subtask plan (from Phase 2)
  - Test files to make pass (from Phase 3)
  - Project standards reference
RECEIVE:
  - Implementation complete
  - Tests passing (verified)
  - Files modified list
```

**After delegation returns:**
- Verify all tests pass

### 4.2 Pre-Commit Review (Conditional)

```
IF files_changed >= 4 OR user_requests_review:
  DELEGATE to subagents/code/reviewer
  PASS:
    - Files changed (from git diff --name-only)
    - Implementation context
  RECEIVE:
    - Review notes
    - Risk level
    - Suggested improvements (do not auto-apply)

  Present review notes to user.
  IF risk_level == "HIGH": Require explicit approval to proceed.
```

### 4.3 Commit Implementation

**Commit implementation:**
  ```
  fix|feat: [description] for issue #[number]
  
  [What was implemented and why]
  ```

## Phase 5: Validate & Complete

### 5.1 Build Validation
```
DELEGATE to subagents/code/build-agent
RECEIVE: Type check + build status
```
If FAIL: Stop, report errors, fix before proceeding.

### 5.2 Security Audit (Mandatory)
```
DELEGATE to subagents/code/security-auditor
PASS:
  files_changed: [from git diff]
  base_ref: [branch point or HEAD~n]
RECEIVE:
  recommendation: PASS | BLOCK | REVIEW
  findings: [severity-rated issues]
  dependency_issues: [if any]
```

**Handle recommendation:**

| Result | Action |
|--------|--------|
| PASS | Continue to 5.3 |
| REVIEW | Present findings, ask user to confirm proceed or fix |
| BLOCK | Stop. Present critical issues. Options: fix and re-run, override with justification (logged), or abort |

### 5.3 Push & Complete

**For direct commits:**
```bash
git push origin master
gh issue close [number] --comment "Fixed in [commit hash]

Changes: [summary]
Verified: [how tested]"
```

**For branch + PR:**
```bash
git push -u origin [branch-name]
gh pr create \
  --title "Fix #[number]: [description]" \
  --body "Fixes #[number]

## Changes
- [change list]

## Testing
- [test results]

## Security
- [audit result: PASS or findings addressed]"
```

Present completion:
```
Issue #[number] completed!

Workflow: [Direct | Branch + PR]
Commits: [count] | Files: [count]
Tests: [pass count] | Security: [PASS/findings addressed]
PR: [URL if applicable]
```

## Error Handling

| Error | Response |
|-------|----------|
| `gh` not installed | "Install GitHub CLI: https://cli.github.com" |
| Not authenticated | "Run `gh auth login`" |
| Issue not found | "Issue #[n] not found. Check number." |
| Tests fail | Stop, help fix. Don't proceed until green. |
| Build fails | Stop, report errors. Don't proceed until fixed. |
| Security BLOCK | Stop, present findings. Require fix or explicit override. |
| Branch exists | Ask: use existing or create new? |

## Subagent Delegation Summary

| Phase | Subagent | Trigger | Required |
|-------|----------|---------|----------|
| 2 | task-manager | 4+ files OR complex | Conditional |
| 3 | tester | Always | Yes |
| 4.1 | coder-agent | Always | Yes |
| 4.2 | reviewer | 4+ files OR user request | Conditional |
| 5.1 | build-agent | Always | Yes |
| 5.2 | security-auditor | Always | Yes |
