# Git to Jujutsu Differences

Comprehensive comparison between Git and Jujutsu commands, concepts, and mental models.

---

## Core Philosophy Differences

| Aspect | Git | Jujutsu (jj) |
|--------|-----|--------------|
| **Staging** | Explicit staging area (`git add`) | No staging - all changes auto-tracked |
| **Branch** | Branch (named pointer) | Bookmark (named pointer for remote) |
| **HEAD** | Current branch reference | Working copy commit (`@`) |
| **Commit** | Snapshot of staged files | Commit of all tracked files |
| **History** | Immutable (once pushed) | Rewritable (change IDs stable) |
| **Undo** | Limited (`reflog`) | Full operation log |
| **Conflicts** | Blocking state | Structured data, non-blocking |

---

## Mental Model: Working Copy as Commit

### Git: Staging Area Model
```
Working Directory → Staging Area → Repository
     git add           git commit
```

### Jujutsu: Direct Commit Model
```
Working Directory → Working Copy Commit (@)
     (auto-tracked)    (always a real commit)
```

In jj, `@` (working copy) is ALWAYS a commit with:
- A description (commit message)
- A change ID (stable)
- A commit ID (volatile, based on content)
- Files (tracked)

You never "stage" - changes are immediately part of `@`.

---

## Quick Reference Table

| Git Command | Jujutsu Equivalent | Notes |
|-------------|---------------------|-------|
| `git status` | `jj st` | Shows working copy |
| `git status -s` | `jj st --quiet` | Short format |
| `git log` | `jj log` | |
| `git log --oneline` | `jj log --no-graph` | |
| `git diff` | `jj diff` | |
| `git diff --staged` | N/A | No staging in jj |
| `git diff HEAD~1` | `jj diff -r @-` | |
| `git show <ref>` | `jj show <rev>` | |
| `git blame` | `jj file annotate` | |
| `git add` | (none needed) | Auto-tracked |
| `git commit` | `jj commit -m "msg"` | |
| `git commit -a` | `jj commit -m "msg"` | `-a` not needed |
| `git commit --amend` | `jj describe -m "msg"` + edit | |
| `git checkout <ref>` | `jj edit <rev>` | |
| `git checkout -b <name>` | `jj new -m "msg"` | + bookmark if needed |
| `git branch` | `jj bookmark list` | |
| `git branch <name>` | `jj bookmark create <name>` | |
| `git branch -d <name>` | `jj bookmark delete <name>` | |
| `git merge <ref>` | `jj new <a> <b>` | Create merge commit |
| `git rebase <base>` | `jj rebase -s <src> -d <dest>` | |
| `git merge --squash` | `jj squash` | |
| `git reset --soft` | `jj undo` | |
| `git reset --hard` | **AVOID** | Use `jj undo` |
| `git stash` | `jj new -m "temp"` | Create temp commit |
| `git stash pop` | `jj edit <stash>` | |
| `git fetch` | `jj git fetch` | |
| `git pull` | `jj git fetch && jj rebase -d main@origin` | |
| `git push` | `jj git push` | |
| `git push -f` | N/A | jj has no force push |
| `git worktree add` | `jj workspace add` | |
| `git reflog` | `jj op log` | Better! |
| `git revert <ref>` | `jj new -m "Revert..."` | Manual |

---

## Detailed Command Mapping

### Viewing State

#### Status
```bash
# Git
git status
git status -s
git status --short

# Jujutsu
jj st
jj st --quiet
jj status
```

#### Log
```bash
# Git
git log
git log --oneline
git log -n 10
git log --author="name"
git log --grep="pattern"
git log -p
git log --stat

# Jujutsu
jj log
jj log --no-graph
jj log -n 10
jj log -r 'author(name)'
jj log -r 'description.regex("pattern")'
jj log -p
jj log --stat
```

#### Diff
```bash
# Git
git diff
git diff --staged
git diff HEAD~1
git diff main..feature

# Jujutsu
jj diff
jj diff -r @-                    # vs parent
jj diff -r main..feature         # range syntax
jj diff --git                    # git-compatible
```

#### Show/Blame
```bash
# Git
git show <ref>
git blame <file>
git blame -L 10,20 <file>

# Jujutsu
jj show <rev>
jj file annotate <file>
jj file annotate -L 10,20 <file>
```

---

### Creating & Editing Commits

#### New Commit
```bash
# Git
git commit -m "message"
git commit -am "message"    # add + commit

# Jujutsu
jj commit -m "message"     # describe + new in one
jj new -m "message"        # create and switch to it
jj describe -m "message"   # update message
```

**Key difference:** In jj, you typically:
1. Create a commit with `jj new -m "msg"` (becomes working copy)
2. Make changes
3. Squash with `jj squash` when done

#### Amend Commit Message
```bash
# Git
git commit --amend -m "new message"

# Jujutsu
jj describe -m "new message"
```

#### Amend Content
```bash
# Git
# Edit files
git add .
git commit --amend --no-edit

# Jujutsu
# Edit files (already tracked)
# jj auto-saves - no explicit amend needed
```

---

### Navigating History

#### Checkout/Switch
```bash
# Git
git checkout <branch>
git checkout <commit>
git checkout -b <new-branch>

# Jujutsu
jj edit <rev>                 # Edit commit (switch WC to it)
jj new -m "message"           # Create new, switch to it
jj new <base>                # Create from base, switch
jj new --no-edit <base>      # Create without switching
```

#### Moving Between Parents
```bash
# Git
git checkout HEAD~1
git checkout main~3

# Jujutsu
jj edit @-
jj edit @-3
```

---

### Branches (Bookmarks)

#### List
```bash
# Git
git branch
git branch -a
git branch -r

# Jujutsu
jj bookmark list
jj bookmark list --all
jj bookmark list --remote
```

#### Create
```bash
# Git
git branch <name>
git checkout -b <name>

# Jujutsu
jj bookmark create <name>
jj bookmark set <name>        # Move to current @ 
```

#### Delete
```bash
# Git
git branch -d <name>
git branch -D <name>          # force

# Jujutsu
jj bookmark delete <name>
```

#### Rename
```bash
# Git
git branch -m <old> <new>

# Jujutsu
jj bookmark rename <old> <new>
```

---

### Merging & Rebasing

#### Merge
```bash
# Git
git merge <branch>
git merge --no-ff <branch>

# Jujutsu
jj new <main> <feature>       # Create merge commit
jj merge <branch>             # v0.36+ experimental
```

#### Rebase
```bash
# Git
git rebase <base>
git rebase -i <base>
git rebase --onto <new> <old> <branch>

# Jujutsu
jj rebase -s <source> -d <dest>
jj rebase -s <source> -o <dest>    # v0.36+ syntax
# Note: jj rebase is ALWAYS non-interactive
```

#### Squash
```bash
# Git
git merge --squash <branch>
git rebase -i (squash in editor)

# Jujutsu
jj squash                     # Into parent
jj squash -into <target>      # Into specific commit
```

#### Split Commit
```bash
# Git
git reset HEAD~
# Stage hunks individually
# git commit

# Jujutsu
jj duplicate <rev>            # SAFE: duplicate first
jj split -r <duplicate> <paths>  # By files
```

---

### Undo & Recovery

#### Reset
```bash
# Git
git reset --soft HEAD~1
git reset --mixed HEAD~1
git reset --hard HEAD~1

# Jujutsu
jj undo                      # BEST: undo operation
jj restore <paths>            # Discard WC changes
jj restore -r <rev> <paths>   # Restore from revision
```

#### Reflog/Operation Log
```bash
# Git
git reflog
git reflog show HEAD@{2}

# Jujutsu
jj op log
jj op log --no-graph
jj --at-op <id> st
jj op restore <id>            # DANGEROUS - ask user first!
```

---

### Remote Operations

#### Fetch
```bash
# Git
git fetch
git fetch <remote>
git fetch --all

# Jujutsu
jj git fetch
jj git fetch --remote <name>
jj git fetch --all
```

#### Pull
```bash
# Git
git pull
git pull --rebase

# Jujutsu
jj git fetch
jj rebase -d main@origin
```

#### Push
```bash
# Git
git push
git push <remote> <branch>
git push -u <remote> <branch>
git push --force

# Jujutsu
jj git push
jj git push --bookmark <name>
jj git push --bookmark <name> --remote <remote>
# No force push - jj prevents this!
```

---

### Worktrees (Workspaces)

#### Add
```bash
# Git
git worktree add <path> <branch>

# Jujutsu
jj workspace add <path>
jj workspace add --name <name> -r <rev> <path>
```

#### List
```bash
# Git
git worktree list

# Jujutsu
jj workspace list
```

#### Remove
```bash
# Git
git worktree remove <path>

# Jujutsu
jj workspace forget <name>    # Keeps files
jj workspace delete <name>   # Deletes (rarely needed)
```

---

### Stashing

```bash
# Git
git stash
git stash push -m "message"
git stash list
git stash pop
git stash apply

# Jujutsu
jj new -m "temp: message"     # Create temp commit
jj edit <stash-rev>          # Return to it
# No stash needed - jj makes it easy to create commits!
```

---

## Concepts: Git → Jujutsu

### Change ID vs Commit ID

| Aspect | Change ID | Commit ID |
|--------|-----------|-----------|
| Stability | Stable (doesn't change) | Volatile (changes on rewrite) |
| Like | Git refs (branches) | Git SHA |
| Use for | User references | Internal only |
| Example | `qzvzulzz` | `abc123def456...` |

**Rule:** Always use change IDs in conversation/commands.

### Bookmarks vs Branches

| Aspect | Bookmark | Branch |
|--------|----------|--------|
| Purpose | Remote sync | Daily reference |
| Local default | Usually don't create | Create freely |
| Protection | Immutable by default | Mutable |
| Push | Required | Optional |

In jj, you typically:
- Don't create bookmarks for local work
- Use change IDs to reference commits
- Create bookmarks only when ready to push

### Conflicts as State

| Aspect | Git | Jujutsu |
|--------|-----|---------|
| During rebase | Blocked until resolved | Continues, stores conflict |
| Resolution | Editor-based | `jj resolve` |
| State | "Rebase in progress" | Normal commit state |

In jj, conflicts are stored as structured data in the commit. You can:
- Continue working
- View conflicts with `jj st`
- Resolve with `jj resolve`
- Create resolution commits

---

## Configuration Equivalents

```bash
# Git config
git config user.name "Name"
git config user.email "email"

# Jujutsu config
jj config set user.name "Name"
jj config set user.email "email"

# Or in ~/.jjconfig.toml
[user]
name = "Name"
email = "email"
```

---

## Common Workflow Translation

### Daily Development

```bash
# Git
git checkout -b feature
# make changes
git add .
git commit -m "feat: add feature"
git push

# Jujutsu
jj new -m "feat: add feature"
# make changes
jj squash
# repeat until done
jj git push
# (jj auto-creates bookmark on push if needed)
```

### Update with Main

```bash
# Git
git checkout main
git pull
git checkout feature
git rebase main
git push --force

# Jujutsu
jj git fetch
jj rebase -s @ -d main@origin
jj git push
# No force push needed!
```

---

## What Git Concepts Don't Apply

1. **Staging** - No concept in jj
2. **Force push** - Doesn't exist (by design)
3. **Detached HEAD** - Working copy is always attached
4. **Merge conflicts blocking** - Stored as data
5. **SHA changes** - Change IDs are stable
6. **Reflog expiry** - Operation log is permanent

---

*For command reference, see `references/commands.md`. For revset syntax, see `references/revsets.md`.*
