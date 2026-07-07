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
| `git rebase <base>` | `jj rebase -s <src> -o <dest>` | `-d` is alias for `-o` |
| `git merge --squash` | `jj squash` | |
| `git reset --soft` | `jj undo` | |
| `git reset --hard` | **AVOID** | Use `jj undo` |
| `git stash` | `jj new -m "temp"` | Create temp commit |
| `git stash pop` | `jj edit <stash>` | |
| `git fetch` | `jj git fetch` | |
| `git pull` | `jj git fetch && jj rebase -o main@origin` | |
| `git push` | `jj git push` | |
| `git push -f` | N/A | jj has no force push |
| `git worktree add` | `jj workspace add` | |
| `git reflog` | `jj op log` | Better! |
| `git revert <ref>` | `jj revert <rev>` | Reverse a revision's changes |

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
jj rebase -s @ -o main@origin
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
