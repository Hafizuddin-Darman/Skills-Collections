---
name: jujutsu-jj-vcs
description: Jujutsu (jj) version control system mastery. Use when working with jj, jujutsu, commits, branches, bookmarks, rebasing, splitting, workspaces, or any VCS operations. Handles basic operations and routes to references for advanced workflows. Covers jj syntax (@, change-id, revsets), Git differences, staged commits, parallel agents, and recovery.
---

# Jujutsu (jj) VCS

A comprehensive skill for working with Jujutsu (jj), a Git-compatible version control system that provides a simpler mental model and powerful history editing.

**IMPORTANT:** Jujutsu is NOT Git. Syntax and mental model differ significantly.

---

## Core Philosophy (READ FIRST)

1. **Working Copy = Commit** - There's no staging area. The working copy (`@`) is always a real commit with a description. All changes are auto-tracked.

2. **Change IDs are Stable** - Change IDs (e.g., `qzvzulzz`) don't change when you rewrite history. Commit IDs do. Always use change IDs for references.

3. **Conflicts are State, Not Emergencies** - Rebases succeed even with conflicts. Conflicts are stored as structured data, not blocking state.

4. **Operation Log is Your Safety Net** - Every operation is recorded. `jj op log` lets you undo anything, ever.

5. **Bookmarks Exist for GitHub** - Use bookmarks for pushing to remotes, but your daily workflow uses change IDs.

---

## jj-Specific Keywords & Symbols

| Symbol | Meaning | Example |
|--------|---------|---------|
| `@` | Current working copy | `jj log -r @` |
| `@-`, `@--` | Parent, grandparent of working copy | `jj squash --into @-` |
| `@+` | Child of working copy | `jj new @+` |
| `x::y` | Range: ancestors of x through descendants of y (inclusive) | `main::` |
| `x..y` | Range: descendants of x excluding y | `main..@` |
| `x\|y` | Union: commits in either x or y | `author(tom)\|author(ann)` |
| `x&y` | Intersection: commits in both x and y | `main&@` |
| `x~y` | Difference: commits in x but not in y | `@~3` (3 parents back) |
| `root()` | The root commit (empty) | `root()::` (all commits) |
| `trunk()` | The trunk/main line | `trunk()` |
| `change-id` | Stable ID - USE THESE | `qzvzulzz` |
| `commit-id` | Volatile ID - AVOID | `abc123def` |

### Common Revset Functions

| Function | Returns | Example |
|----------|---------|---------|
| `parents(x)` | Parents of x | `parents(@)` |
| `children(x)` | Children of x | `children(main)` |
| `ancestors(x)` | All ancestors of x | `ancestors(@)` |
| `descendants(x)` | All descendants of x | `descendants(main)` |
| `heads(x)` | Heads of the set | `heads(all())` |
| `bookmarks()` | All bookmarks | `bookmarks()` |
| `mine()` | Your commits | `mine()` |
| `conflicted()` | Commits with conflicts | `conflicted()` |
| `empty()` | Empty commits (TODOs) | `empty()` |

---

## Quick Comparison: Git vs Jujutsu

| Concept | Git | Jujutsu (jj) |
|---------|-----|--------------|
| **Staging** | Staging area (`git add`) | None - all changes auto-tracked |
| **Branch** | Branch | Bookmark |
| **Checkout** | `git checkout` | `jj edit` (or `jj new` for new work) |
| **Commit** | `git commit` | `jj commit -m "message"` or `jj new` + edit |
| **Amend** | `git commit --amend` | `jj describe -m "message"` + edit files |
| **Squash** | `git merge --squash` | `jj squash` |
| **Split commit** | Manual | `jj split -r <rev>` |
| **Rebase** | `git rebase` | `jj rebase -s <src> -d <dest>` |
| **Stash** | `git stash` | `jj new` (create temp commit) |
| **Blame** | `git blame` | `jj file annotate` |
| **Undo** | Limited | `jj undo`, `jj op log` (full history!) |
| **Worktree** | `git worktree` | `jj workspace add` |
| **HEAD** | Current branch | `@` (working copy commit) |

**Read `references/git-differences.md` for comprehensive mapping →**

---

## Essential Commands (Daily Use)

### Viewing State
```bash
jj st           # Status - shows working copy
jj log          # Commit log (use -r to filter)
jj diff         # Changes in working copy
jj diff -r <rev>  # Changes in a revision
jj show <rev>   # Show commit content
jj file show -r <rev> <path>  # Show file at revision
```

### Creating & Editing Commits
```bash
jj new -m "message"           # Create new commit (stays on it as working copy)
jj describe -m "message"      # Update commit message (shorthand: jj desc)
jj edit <rev>                 # Edit existing commit (switch working copy to it)
jj commit -m "message"        # Describe current + create new working copy in one
jj new --no-edit <base>       # Create without switching to it
```

### Restructuring History
```bash
jj squash                    # Move all changes into parent commit
jj squash -m "message"       # Squash with custom message
jj absorb                   # Auto-distribute changes to ancestor commits
jj split -r <rev> <paths>   # Split commit by file paths
jj rebase -s <src> -o <dest>    # Rebase source onto destination
jj rebase -r <rev> -o <dest>    # Rebase single revision (no descendants)
jj duplicate <rev>          # Create copy (SAFE before destructive ops)
```

### Bookmarks (Branches)
```bash
jj bookmark list            # List bookmarks
jj bookmark create <name>  # Create bookmark
jj bookmark set <name>     # Move bookmark to working copy
jj bookmark delete <name>  # Delete bookmark
jj bookmark track <name>@origin  # Track remote bookmark
```

### Remote Operations
```bash
jj git fetch               # Fetch from remote
jj git push                # Push all
jj git push --bookmark <name>  # Push specific bookmark
jj git push --bookmark <name> --remote <remote>
```

### Navigating

```bash
jj next                       # Move to child revision
jj prev                       # Move to parent revision
```

### Reverting

```bash
jj revert <rev>               # Reverse a revision's changes
```

### Evolog

```bash
jj evolog                     # Show how a change evolved over time
```

### Recovery & Safety (CRITICAL)
```bash
jj undo                    # Undo last operation
jj op log                  # View operation history
jj op log --no-graph       # Flat list
jj --at-op=<id> st        # Check state at operation
jj op restore <id>        # Restore to operation (ask user first!)
jj restore <paths>        # Discard changes in working copy
jj restore --from <src> <paths>  # Restore files from another revision
```

### Workspaces
```bash
jj workspace add <path>    # Create new workspace
jj workspace list          # List all workspaces
jj workspace forget <name> # Remove workspace (keeps files)
jj workspace root          # Show workspace root
jj workspace update-stale # Update to latest
```

**Read `references/commands.md` for complete command documentation →**

---

## Decision Tree → Use References

| User Goal | Reference to Read |
|-----------|------------------|
| All available commands | `references/commands.md` |
| Selection syntax (revsets, filesets, templates) | `references/revsets.md` |
| Comprehensive Git to jj mapping | `references/git-differences.md` |
| Detailed workflow patterns | `references/workflows.md` |
| Recovery, safety, undo procedures | `references/recovery.md` |

---

## LLM-Specific Rules (IMPORTANT)

### ✅ ALWAYS Do
- Use `-m "message"` flag for all commands that accept it
- Use change IDs (stable), not commit IDs (volatile)
- Create dedicated workspace per parallel task
- Fetch before push: `jj git fetch` then `jj git push`
- Use `jj duplicate` before destructive operations

### ❌ NEVER Do
- Use interactive commands: `jj split -i`, `jj diffedit`
- Use `git reset --hard` (use `jj undo` instead)
- Share workspace paths between agents
- Force push (jj doesn't have force push - that's good!)
- Skip the `-m` flag - it blocks on editor

### Safety Rules for Multi-Agent
- One workspace per agent: `jj workspace add --name <agent>-<task>`
- Use change IDs for coordination (stable across rebases)
- Never abandon shared commits
- Always fetch before rebasing: `jj git fetch && jj rebase ...`
- Read `references/workflows.md` for parallel agent patterns

---

## Workflow Patterns

### Pattern 1: Staged Commit (Recommended)

Best for keeping work organized:

```bash
# 1. Create placeholder commit (the "stage")
jj new -m "feat: user authentication"

# 2. Create working copy on top
jj new

# 3. Make changes, test

# 4. Squash into placeholder
jj squash

# 5. Repeat steps 2-4 until feature complete

# 6. Push
jj git push
```

### Pattern 2: Edit Existing Commit

Fix a commit in the stack:

```bash
# 1. Navigate to the commit
jj edit <change-id>

# 2. Make changes

# 3. jj auto-saves (no explicit save needed)
#    The changes are part of the working copy

# 4. Move back to original position if needed
jj squash --into <original-commit>
```

### Pattern 3: Parallel Agents (Colocated)

For multi-agent AI environments:

```bash
# Agent A:
jj workspace add --name myproject-agentA-featureA -r main@origin ../myproject-agentA-featureA

# Agent B:
jj workspace add --name myproject-agentB-featureB -r main@origin ../myproject-agentB-featureB

# Each agent works independently in their workspace

# When done, coordinate via change IDs:
jj bookmark set pr/agentA/featureA -r <change-id>
jj git push --bookmark pr/agentA/featureA
```

### Pattern 4: Safe Split (Duplicate First)

Always duplicate before splitting:

```bash
# 1. Duplicate the commit (safety copy)
jj duplicate <rev>

# 2. Split the duplicate
jj split -r <duplicate-rev> <paths-to-first-changeset>

# 3. Verify with jj interdiff
jj interdiff --from <original> --to <split-result>

# 4. If good, abandon original
jj abandon <original-rev>
```

### Pattern 5: Conflict Resolution

```bash
# 1. After rebase/split shows conflicts
jj resolve --list    # See conflicted files

# 2. Open and edit the files
#    jj stores conflicts as <<<<<<< markers

# 3. Mark as resolved
jj resolve <path>

# 4. Create resolution commit
jj new -m "Resolve conflict in file.rs"

# 5. Squash into the conflict commit if desired
jj squash --into <conflict-commit>
```

---

## Common Error Recovery

| Problem | Solution |
|---------|----------|
| Accidental edit | `jj undo` |
| Wrong commit message | `jj describe -m "correct message"` |
| Lost commits | `jj op log` → find → `jj op restore <id>` |
| Wrong branch | `jj bookmark set <correct> -r @` |
| Stuck in editor | Check for running jj processes, kill if needed |
| Corrupted repo | `jj op log` usually recovers |

**Read `references/recovery.md` for detailed recovery procedures →**

---

## Reading References for More Detail

This skill provides the essentials. For comprehensive documentation, read these files:

1. **`references/commands.md`** - Complete jj command reference with all flags
2. **`references/revsets.md`** - Full revset syntax, filesets, and templates
3. **`references/git-differences.md`** - Detailed side-by-side Git vs jj for every operation
4. **`references/workflows.md`** - Staged commits, stacked PRs, parallel agents, TODO workflow
5. **`references/recovery.md`** - Operation log deep dive, safety rules, emergency procedures

---

## Related Skills

This skill works well with:
- Skills for specific languages/frameworks (jj is language-agnostic)
- Code review skills (after `jj git push`)
- CI/CD skills (for automated pipelines)

---

*This skill is designed for AI agents working with Jujutsu (jj) version control. Jujutsu provides a safer, more collaborative alternative to Git, especially for AI-assisted development.*
