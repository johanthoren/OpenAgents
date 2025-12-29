---
description: Work on GitHub issues with guided workflow from selection to completion
---

# Work on GitHub Issue

You are an AI agent that helps work on GitHub issues using a structured workflow. Follow these instructions to guide the user from issue selection through implementation and completion.

## Instructions for Agent

When the user runs this command, execute the following workflow:

### Phase 1: Issue Selection

**Check $ARGUMENTS for issue number:**

Parse `$ARGUMENTS` to detect an issue number. Valid formats:
- `42` - plain number
- `#42` - with hash prefix
- `issue-42` - with prefix
- Any string containing digits that represent an issue ID

If a valid issue number is found in `$ARGUMENTS`:
- Extract the numeric ID
- Skip to Phase 2

**If no issue number provided:**

Show recent open issues:
```bash
gh issue list --state open --limit 10 --json number,title,labels,updatedAt
```

Present to user (using issue IDs only, no separate list index):
```
Recent open issues:

#42: Fix authentication bug [bug, authentication] (updated 2 days ago)
#38: Add dark mode theme [enhancement, ui] (updated 3 days ago)
#35: Improve error messages [enhancement] (updated 1 week ago)
...

Enter an issue ID (e.g., 42 or #42) to work on it, or:
- A search query to find other issues
- 'q' to quit

>
```

**Note**: Always use the GitHub issue ID to select an issue, whether it appears in the list above or not. Any valid issue ID from the repository can be used.

**If user provides search query:**
```bash
gh issue list --search "[query]" --state open --limit 10 --json number,title,labels
```

Show results and repeat selection prompt.

### Phase 2: Fetch Issue Context

Get full issue details:
```bash
gh issue view [number] --json number,title,body,state,labels,assignees,comments
```

Check for existing scratchpad:
```bash
find scratchpads -type f -name "issue-[number]-*.md" 2>/dev/null
```

Check for related PRs:
```bash
gh pr list --search "[number]" --state all --json number,title,state
```

Present context to user:
```
Issue #[number]: [Title]
State: [open/closed]
Labels: [labels]
Assignees: [assignees or "unassigned"]

[Issue body]

Context:
- Scratchpad: [path or "none found"]
- Related PRs: [count] ([states])
- Comments: [count]
```

### Phase 3: Understand & Plan

Ask user for clarification:
```
Before we start, let's clarify:

1. Is the issue description clear and complete? (y/n)
   > [User input]

2. Are there any constraints or requirements not mentioned? (or 'none')
   > [User input]

3. Should we work directly on master or create a branch? (master/branch)
   > [User input]
```

**Branch naming convention:**
- feature/issue-[number]-[brief-description]
- fix/issue-[number]-[brief-description]
- refactor/issue-[number]-[brief-description]

**Workflow decision:**

Use **direct commit to master** when:
- Single logical change
- Low risk (docs, config, minor fixes)
- Few files affected (1-3 files)
- Quick implementation (<30 min)

Use **branch + PR workflow** when:
- Multiple logical changes needed
- Complex implementation
- Many files affected (4+ files)
- Requires review or testing
- Breaking changes

**CRITICAL**: Never squash complex changes into a single commit to avoid branching. Each commit must be:
- Independently understandable
- Focused on a single logical change
- Small enough to review easily

If you're tempted to create a large commit, that's a signal you need multiple commits and therefore a branch/PR.

### Phase 4: Create Implementation Plan

Analyze what needs to be done:
```
Based on the issue, here's my implementation plan:

Files to modify:
- [file1]: [what changes]
- [file2]: [what changes]

Estimated commits: [number]
Estimated time: [time]

Does this approach make sense? (y/n/suggest changes)
> [User input]
```

If user suggests changes, incorporate feedback and re-present plan.

**Update scratchpad** with plan:

If scratchpad exists:
```bash
cat >> scratchpads/issue-[number]-*.md << 'EOF'

## Implementation Plan ($(date +%Y-%m-%d))

### Approach
[Detailed approach]

### Files to Change
- [file1]: [changes]
- [file2]: [changes]

### Steps
1. [Step 1]
2. [Step 2]
...

### Testing Strategy
- [How to verify]

EOF
```

If no scratchpad, offer to create one:
```
Would you like to create a scratchpad for this work? (y/n)
```

**IMPORTANT**: Follow scratchpad guidelines:
- DO NOT include personal data, API keys, passwords, or sensitive information
- Assume public visibility - these are committed to git history
- Focus on technical analysis, design decisions, and rationale

### Phase 5: Create Branch (if needed)

If using branch workflow:
```bash
git checkout -b [branch-name]
```

Confirm:
```
Created branch: [branch-name]
Ready to implement.
```

### Phase 6: Implementation

**For each logical change:**

1. **Implement the change**
   - Read relevant files
   - Make necessary modifications
   - Follow project coding standards from @AGENTS.md
   - Keep functions small (5-15 lines)
   - Use clear, descriptive names

2. **Verify the change**
   - Run relevant tests
   - Check for syntax errors
   - Verify behavior matches requirements
   - Follow verification workflow from AGENTS.md

3. **Commit the change**
   - Stage relevant files
   - Use `/commit` command for well-formatted commit message
   - Or create commit manually with conventional format

**Example implementation flow:**
```
Step 1: [Description]
- Modified: [files]
- Changes: [what changed]
- Verified: [how verified]
- Committed: [commit hash]

Step 2: [Description]
...
```

### Phase 7: Testing

Run project tests:
```bash
# Detect test framework and run tests
npm test || cargo test || go test ./... || pytest || bundle exec rspec
```

Present results:
```
Test Results:
- Total: [count]
- Passed: [count]
- Failed: [count]

[If failures, show details]
```

If tests fail:
- Analyze failures
- Determine if related to changes
- Fix issues and rerun tests
- Don't proceed until tests pass

**Manual verification** (if applicable):
- Run the application
- Test the specific feature/fix
- Verify behavior matches success criteria
- Document observations

### Phase 8: Completion

**For direct commits to master:**
```bash
git push origin master
```

Close the issue:
```bash
gh issue close [number] --comment "Fixed in commit [hash]

Changes:
- [Summary of changes]

Verified:
- [How verified]"
```

**For branch + PR workflow:**

Push branch:
```bash
git push -u origin [branch-name]
```

Create PR:
```bash
gh pr create \
  --title "Fix #[number]: [brief description]" \
  --body "Fixes #[number]

## Changes
- [Change 1]
- [Change 2]

## Testing
- [How tested]
- [Test results]

## Verification
- [How verified]" \
  --label "[appropriate labels]"
```

Present PR details:
```
✅ Pull request created!

PR #[number]: [Title]
URL: [GitHub URL]

The PR is ready for review.
```

### Phase 9: Summary & Documentation

Update scratchpad with resolution:
```bash
cat >> scratchpads/issue-[number]-*.md << 'EOF'

## Resolution ($(date +%Y-%m-%d))

### Implementation Summary
[What was implemented]

### Commits
- [commit 1]: [description]
- [commit 2]: [description]

### Testing
[Test results and verification]

### PR
[PR link if applicable]

### Learnings
[Any insights or decisions made]

EOF
```

Present comprehensive summary:
```
✅ Issue #[number] completed!

Workflow: [Direct commit | Branch + PR]
Branch: [branch name if applicable]
Commits: [count]
Files changed: [count]
Tests: [passed/failed]
PR: [URL if applicable]

Changes:
- [Summary of what was done]
- [Key decisions made]

Verification:
- [What was tested]
- [Results observed]

Next steps:
- [If PR: Wait for review and merge]
- [If direct: Issue is closed and complete]
```

## Best Practices

**Planning**:
- Understand the issue fully before starting
- Break complex issues into logical steps
- Estimate time realistically
- Identify dependencies early

**Implementation**:
- Make small, focused commits
- Test continuously, not just at the end
- Follow project coding standards
- Document non-obvious decisions

**Verification**:
- Run tests after each change
- Manually verify behavior
- Check edge cases
- Don't declare success without verification

**Communication**:
- Update issue with progress
- Document decisions in scratchpad
- Write clear commit messages
- Provide thorough PR descriptions

## Error Handling

- If `gh` not installed: "GitHub CLI required. Install from: https://cli.github.com"
- If not authenticated: "Run 'gh auth login' to authenticate with GitHub"
- If issue not found: "Issue #[number] not found. Check the issue number."
- If tests fail: Don't proceed, help user fix failures
- If branch already exists: Ask user if they want to use existing or create new

## Integration with OpenAgents

This command integrates with:
- `/commit` - Well-formatted commits with emoji
- `/create-issue` - Issue creation workflow
- Task breakdown workflows - Complex issue handling
- Code review processes - Quality gates
- Testing workflows - Verification

## Notes

- Always verify changes before committing
- Follow the verification workflow from AGENTS.md
- No "should work" - actually run and verify
- Keep commits atomic and focused
- Update documentation as you go
- Communicate progress clearly
