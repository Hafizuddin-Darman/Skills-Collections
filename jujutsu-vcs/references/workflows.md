# Jujutsu Workflows

Detailed workflow patterns for common development scenarios with Jujutsu (jj).

---

## 1. Staged Commit Workflow

Recommended for keeping work organized. Creates a "placeholder" commit that you build into.

### Steps

```bash
# 1. Create placeholder (stage) commit
jj new -m "feat: user authentication"

# 2. Create working copy on top
jj new

# 3. Make changes to your code
# ... edit files ...

# 4. Squash changes into placeholder
jj squash

# 5. Repeat steps 2-4 until feature is complete
#    Each cycle: jj new, make changes, jj squash

# 6. Verify and push
jj log -r @-::        # See your stack
jj git push
```

### Why This Works

- Placeholder keeps your intention clear
- Working copy (`@`) is always a real commit
- Squash compresses work into clean commits
- Easy to see what you've done with `jj log @-::`

### Variations

**Quick single commit:**
```bash
jj commit -m "feat: add feature"
# Equivalent to describe + new in one
```

**Preserve detailed history:**
```bash
jj new -m "feat: start auth"
jj new
# make initial changes
jj squash
jj new  
# make more changes  
jj squash
# Result: multiple commits
```

---

## 2. Edit Existing Commit

Fix a commit in the middle of your stack without rebasing everything.

### Steps

```bash
# 1. Navigate to the commit you want to fix
jj edit <change-id>

# 2. Make your changes
# ... edit files ...

# 3. jj auto-saves changes (no explicit save)

# 4. Optionally squash back to original position
jj squash --into <original-commit>
```

### When to Use

- Fix a typo in an old commit message
- Add a forgotten file
- Fix a bug in a previous commit
- Reorganize your stack

### Important Notes

- Use change IDs (stable), not commit IDs
- `jj edit` switches your working copy to that commit
- All descendants automatically rebase

---

## 3. Safe Split Workflow

Always duplicate before splitting to preserve the original.

### Steps

```bash
# 1. Duplicate the commit (creates safe copy)
jj duplicate <rev>

# 2. Split the duplicate (not the original!)
jj split -r <duplicate-rev> <paths-for-first-part>

# 3. Verify the split
jj interdiff --from <original-rev> --to <split-result>

# 4. If correct, abandon original
jj abandon <original-rev>
```

### Split by Files

```bash
# Files go to first changeset, rest to second
jj split -r <rev> src/main.rs src/lib.rs
```

### Split by Hunk (Manual)

```bash
# For partial file changes:
jj duplicate <rev>              # Safety copy

jj new                          # New WC
jj restore --from <rev> <path> # Restore file

# Manually edit the file to keep only what you want
# ... edit ...

jj describe -m "partial change"  # Describe first part

jj new                          # Next part
# ... rest of changes ...

jj squash --into <first-part>   # Combine
```

---

## 4. Parallel Agents (Colocated)

For multi-agent AI environments where agents work in parallel.

### Setup

```bash
# Each agent creates their own workspace
jj git fetch

# Agent A
jj workspace add --name myproject-agentA-feature -r main@origin ../myproject-agentA-feature

# Agent B  
jj workspace add --name myproject-agentB-feature -r main@origin ../myproject-agentB-feature
```

### Rules for Agents

1. **One workspace per agent** - Never share
2. **Use change IDs** - Stable across rebases
3. **Coordinate via bookmarks** - For PRs:
   ```bash
   jj bookmark set pr/agentA/featureA -r <change-id>
   jj git push --bookmark pr/agentA/featureA
   ```
4. **Always fetch first** - Before any rebase:
   ```bash
   jj git fetch && jj rebase -o main@origin
   ```

### Safety Rules

Destructive-op forbid list (`op restore`, `util gc`, shared bookmark delete, shared abandon) lives in `references/recovery.md`.

### Workspace Naming

```
<repo>-<agent>-<task>
example: myapp-claude-fix-login
```

---

## 5. Stacked PRs Workflow

Multiple PRs stacked on each other, each reviewable independently.

### Creating a Stack

```bash
# Start first PR
jj new main -m "feat: api client"
jj new
# add code
jj squash

# Second PR depends on first
jj new -m "feat: login page"
jj new
# add code  
jj squash

# Third PR depends on second
jj new -m "feat: auth middleware"
jj new
# add code
jj squash
```

### View Stack

```bash
jj log -r main..@ --reversed
```

### Push Stack

```bash
# Each gets its own bookmark
jj bookmark set pr/1-api-client -r <id-1>
jj bookmark set pr/2-login -r <id-2>
jj bookmark set pr/3-auth -r <id-3>

jj git push --bookmark pr/1-api-client --bookmark pr/2-login --bookmark pr/3-auth
```

### Update Stack

```bash
# Fetch latest
jj git fetch

# Rebase entire stack
jj rebase -s <first-in-stack> -d main@origin
# All descendants automatically follow!
```

---

## 6. TODO-as-Commits Workflow

Use empty commits as task markers for planning.

### Status Flags

| Flag | Meaning |
|------|---------|
| `[task:draft]` | Placeholder, needs specification |
| `[task:todo]` | Ready to work |
| `[task:wip]` | In progress |
| `[task:blocked]` | Waiting on external |
| `[task:standby]` | Awaits decision |
| `[task:untested]` | Needs testing |
| `[task:review]` | Ready for review |
| `[task:done]` | Complete |

### Basic TODO Workflow

```bash
# Create TODO (stays on @)
jj new -m "[task:todo] Implement user login"

# Work on it
jj new
# ... implement ...
jj squash

# Mark done
jj describe -m "[task:done] Implement user login"

# Or update flag
jj describe -m "[task:wip] Implement user login"
```

### Parallel TODOs (DAG)

```bash
# Parent task
jj new -m "[task:todo] Build API"

# Child tasks
jj new -m "[task:todo] User endpoint"
jj new
jj squash
# ...
jj describe -m "[task:done] User endpoint"

jj new -m "[task:todo] Auth endpoint"
# ...
```

### Template for TODO Description

```
## Context
Why this task exists

## Requirements
- Specific requirement 1
- Specific requirement 2

## Acceptance Criteria
- [ ] Criterion 1
- [ ] Criterion 2
```

---

## 7. Conflict Resolution Workflow

### After Rebase/Split Shows Conflicts

```bash
# 1. See conflicted files
jj resolve --list

# 2. Open and edit the file
# jj stores conflicts as:
# <<<<<<<
# content from one side
# =======
# content from other side
# >>>>>>>

# 3. Edit to resolve

# 4. Mark as resolved
jj resolve <path>
jj resolve                     # resolve all

# 5. Create resolution commit (optional)
jj new -m "Resolve conflict in file.rs"

# 6. Squash if desired
jj squash -into <conflicted-commit>
```

### Conflict Markers in jj

```jj
<<<<<<<
# Changes at conflict commit
=======
# Changes from rebased source
>>>>>>>
```

---

## 8. Recovery Workflow

Recovery procedures live in `references/recovery.md` — full recovery ladder (undo → op log → at-op → restore), finding lost commits, and emergency `.jj` corruption steps.

---

## 9. Migrate from Git

### Existing Git Repository

```bash
# Option 1: Colocated (recommended)
jj git init --colocate
# jj uses .jj/, Git uses .git/

# Option 2: Pure jj (loses Git history initially)
jj git init
# Can later import: jj git import
```

### Clone to jj

```bash
# Git clone then jj init
git clone <url>
cd repo
jj git init --colocate

# Or: jj can import from Git
jj git import
```

---

## Workflow Selection Guide

| Situation | Recommended Workflow |
|-----------|----------------------|
| Daily development | Staged Commit |
| Fix old commit | Edit Existing |
| Break large commit | Safe Split |
| Multiple agents | Parallel Agents |
| Multiple PRs | Stacked PRs |
| Task planning | TODO-as-Commits |
| Conflicts | Conflict Resolution |
| Made a mistake | Recovery |
| From Git | Migrate from Git |

---

*For command reference, see `references/commands.md`. For recovery procedures, see `references/recovery.md`.*
