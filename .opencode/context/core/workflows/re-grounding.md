<!-- Context: workflows/re-grounding | Priority: high | Version: 1.0 | Updated: 2025-12-17 -->
# Re-grounding Protocol

## Quick Reference

**Purpose**: Establish current state before any task execution - prevents context amnesia

**When to Run**: Session start, session resume (>30min gap), after errors, before major phases

**Protocol**: Read State → Verify Environment → Establish Position → Then Proceed

**Key Insight**: "The mystery of agents is memory" - agents fail when they don't know where they are

---

## Why Re-grounding Matters

From Anthropic's agent research: Long-running agents fail not because models aren't smart enough, but because they start each session without understanding their position in the world.

**Without re-grounding:**
- Agent doesn't know what was already done
- Agent may redo completed work
- Agent may miss failed tasks that need retry
- Agent makes inconsistent decisions

**With re-grounding:**
- Agent knows exactly where it left off
- Agent can pick up failed tasks
- Agent maintains consistent approach
- Agent builds on previous progress

---

## The Re-grounding Protocol

### Step 1: Read State

**Check for existing session:**
- Use the list tool on `.tmp/sessions/` (if missing/empty, report no active session).
- If a session exists, use the read tool on `.tmp/sessions/{session-id}/.manifest.json`.

**Capture git state:**
```bash
# Check repo status first
git rev-parse --is-inside-work-tree

# Current branch and status
git branch --show-current
git status --short

# Recent commits (last 5)
git log --oneline -5

# Uncommitted changes
git diff --name-only
git diff --cached --name-only
```
Only run the branch/status/log/diff commands when inside a git repo; otherwise report "Not a git repository" and continue. When summarizing git status, report only the first 5 lines.

**Check task progress (if task files exist):**
- Use the list tool on `tasks/subtasks/`.
- Use the read tool on `tasks/subtasks/{feature}/objective.md`.

### Step 2: Verify Environment

**Run existing tests (if applicable):**
```bash
# Quick test check - adapt to project
npm test 2>/dev/null || yarn test 2>/dev/null || pytest 2>/dev/null
```
If none are available, report "No test runner found".

**Check build status (if applicable):**
```bash
# Quick build check - adapt to project  
npm run build 2>/dev/null || yarn build 2>/dev/null || cargo check 2>/dev/null
```
If none are available, report "No build system found".

**Identify broken state:**
- Are there failing tests?
- Are there uncommitted changes that might conflict?
- Is there a half-completed task?

### Step 3: Establish Position

**Summarize current state to user:**

```markdown
## Current State

**Git:**
- Branch: `{branch-name}`
- Last commit: `{hash} - {message}` ({time-ago})
- Uncommitted changes: {count} files

**Session:** {session-id or "No active session"}

**Task Progress:** (if applicable)
- ✅ {completed tasks}
- 🔄 {in-progress tasks}
- ❌ {failed tasks needing retry}
- ⏳ {pending tasks}

**Environment:**
- Tests: {passing|failing|not-run}
- Build: {ok|failing|not-checked}

**Ready to proceed?**
```

### Step 4: Then Proceed

Only after re-grounding is complete:
- Move to Stage 1 (Analyze) for new requests
- Resume in-progress task if continuing previous work
- Address failed tasks if any exist

---

## When to Re-ground

Re-ground once per execution chain. Do NOT repeat between sequential tool calls in the same user request unless a trigger applies.

### Always Re-ground

| Trigger | Reason |
|---------|--------|
| New session start | Establish baseline state |
| Session resume (>30min gap) | State may have changed externally |
| After any error/failure | Understand what broke |
| Before major phase transition | Confirm prerequisites met |
| User requests status | Explicit state check |

### Skip Re-grounding

| Trigger | Reason |
|---------|--------|
| Pure informational question | No state dependency |
| Continuation within same task | Already grounded |
| Multiple tool calls within one user request | Re-ground once per execution chain |
| Rapid successive commands (<10 min) with no errors or repo changes | Avoid redundant re-grounding |
| Simple read-only operations | No state changes |

---

## Git State Integration

### Capture on Session Init

When creating a new session, capture git state in manifest:

```json
{
  "session_id": "20250117-143022-a4f2",
  "git_state": {
    "branch": "feature/auth-system",
    "started_at_commit": "a4f2c3d",
    "last_known_commit": "a4f2c3d",
    "commits_this_session": [],
    "initial_status": {
      "staged": [],
      "modified": ["src/auth/login.ts"],
      "untracked": []
    }
  },
  "context_files": { }
}
```

### Update on Commits

After each commit during session:

```json
{
  "git_state": {
    "last_known_commit": "b5e3d4f",
    "commits_this_session": [
      "b5e3d4f - ✨ feat: add login endpoint",
      "a4f2c3d - 🐛 fix: resolve validation issue"
    ]
  }
}
```

### Use for Context

When resuming or re-grounding:
- Compare current HEAD to `last_known_commit`
- If different, external changes occurred - warn user
- Use `commits_this_session` to understand progress
- Search commit messages for task-related keywords

---

## Re-grounding Output Format

### Minimal (for simple tasks)

```
**State:** Branch `main`, clean working tree, no active session.
```
If not in a git repo, report: `Git: unavailable (not a git repository)`.

### Standard (for task continuation)

```markdown
## Current State

**Git:** `feature/auth` @ `a4f2c3d` (2 uncommitted files)
**Session:** `20250117-143022-a4f2` (active)
**Progress:** 3/5 tasks complete, 1 in-progress

Ready to continue with Task 1.3 (login endpoint)?
```

### Detailed (after errors or long gaps)

```markdown
## Current State Summary

**Git State:**
- Branch: `feature/auth-system`
- Last commit: `a4f2c3d - ✨ feat: add user model` (3 hours ago)
- Uncommitted: 2 files
  - `src/auth/login.ts` (modified)
  - `tests/auth.test.ts` (modified)
- External changes: None detected

**Session:** `20250117-143022-a4f2`
- Created: 3 hours ago
- Last activity: 45 minutes ago

**Task Progress:**
- ✅ Task 1.1: Create user model (complete)
- ✅ Task 1.2: Implement password hashing (complete)  
- 🔄 Task 1.3: Create login endpoint (in progress - ~60%)
- ⏳ Task 2.1: Create registration endpoint (pending)
- ⏳ Task 2.2: Add email validation (pending)

**Environment Status:**
- Tests: 12 passing, 2 failing
  - ❌ `test_login_validation` - assertion error
  - ❌ `test_session_expiry` - timeout
- Build: Passing

**Recommended Action:** Fix failing tests before continuing Task 1.3.

Proceed with test fixes, or continue with Task 1.3?
```

---

## Integration with Workflow

Re-grounding is **Stage 0** - runs before the main workflow:

```
Stage 0: Re-ground (when applicable)
    ↓
Stage 1: Analyze
    ↓
Stage 2: Approve
    ↓
Stage 3: Execute
    ↓
Stage 4: Validate
    ↓
Stage 5: Summarize
    ↓
Stage 6: Confirm
```

### Stage 0 Decision Tree

```
Is this a task request (not pure question)?
  ├─ No → Skip to Stage 1
  └─ Yes → Already re-grounded in this request?
           ├─ Yes → Skip (reuse current state)
           └─ No → Check re-ground triggers
                  ├─ New session? → Full re-ground
                  ├─ Resume after gap? → Full re-ground
                  ├─ After error? → Full re-ground
                  ├─ Repo changed? → Full re-ground
                  ├─ Continuing same task? → Skip (already grounded)
                  └─ User requests status? → Full re-ground
```

---

## Best Practices

1. **Re-ground before assuming** - Don't assume you know the state
2. **Git is memory** - Use commit history as progress log
3. **Surface issues early** - Better to discover problems at re-ground than mid-task
4. **Keep summaries scannable** - User should grasp state in 5 seconds
5. **Offer clear next action** - After re-ground, suggest what to do next
6. **Track external changes** - Warn if state changed outside session

---

## Example Re-grounding Flow

```
User: "Continue working on the auth system"

Agent (internal): 
  1. This is a task request → Re-ground needed
  2. Check for active session → Found 20250117-143022-a4f2
  3. Read manifest → Last activity 2 hours ago (gap > 30min)
  4. Capture git state → 2 uncommitted files, same branch
  5. Check task progress → Task 1.3 in progress
  6. Run quick test check → 2 failing tests

Agent (to user):
  ## Current State
  
  **Git:** `feature/auth` @ `a4f2c3d` (2 uncommitted files)
  **Session:** Active (2 hour gap)
  **Progress:** Task 1.3 (login endpoint) in progress
  **Tests:** 2 failing - `test_login_validation`, `test_session_expiry`
  
  **Recommendation:** Address failing tests before continuing.
  
  Options:
  a) Fix failing tests first
  b) Continue Task 1.3 (tests may be WIP)
  c) Show detailed status
```
