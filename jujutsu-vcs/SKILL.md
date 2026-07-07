---
name: jujutsu-jj-vcs
description: Jujutsu (jj) version control system mastery. Use when working with jj, jujutsu, commits, branches, bookmarks, rebasing, splitting, workspaces, or any VCS operations. Handles basic operations and routes to references for advanced workflows. Covers jj syntax (@, change-id, revsets), Git differences, staged commits, parallel agents, and recovery.
---

# Jujutsu (jj) VCS

**IMPORTANT:** Jujutsu is NOT Git. Syntax and mental model differ significantly.

## Core Concepts

1. **Working Copy = Commit** — No staging area. `@` is always a real commit. Changes auto-tracked.
2. **Change IDs are Stable** — `qzvzulzz` survives rewrites. Commit IDs don't. Use change IDs.
3. **Conflicts = State** — Rebases succeed with conflicts. Stored as data, not blocking.
4. **Op Log = Safety Net** — `jj op log` lets you undo anything, ever.

---

## Essential Commands

```bash
jj st                          # Status
jj log -r @                    # Log (filter with -r)
jj new -m "message"            # Create commit
jj describe -m "msg"           # Update message (shorthand: jj desc)
jj commit -m "msg"             # describe + new in one (git-style)
jj edit <rev>                  # Switch working copy to rev
jj squash                      # Move changes to parent
jj absorb                      # Distribute to ancestors
jj split -r <rev> <paths>      # Split commit
jj rebase -s <src> -o <dest>   # Rebase (-d is alias for -o)
jj duplicate <rev>             # Safe copy (before destructive ops)
jj bookmark set <name>         # Move bookmark to @
jj git fetch && jj git push    # Remote ops
jj undo                        # Undo last operation
jj op log                      # Operation history
```

**Read `references/commands.md` for full command docs →**

---

## Committing Work — the "git commit" equivalent

jj has **no save step.** Every command auto-snapshots your working copy into `@`, so changes are captured the moment you make them. You never "snapshot to save" — you *name the current commit and move on.*

Two paths to finish `@` and get a clean working copy. Both land in the same end state: your changes live in `@-` with the message, and `@` is a fresh empty commit.

| Pattern | Commands | Use when |
|---------|----------|----------|
| **describe → new** (primary) | `jj describe -m "msg"` then `jj new` | Still shaping the message, or re-describing before you're happy |
| **commit** (one-shot) | `jj commit -m "msg"` | You want the git-commit feel in a single step |

```bash
# Primary: name the current @, then clear the working copy
jj describe -m "feat: add login"   # set message on current @ (changes already there)
jj new                              # fresh empty @ → your work now lives in @-

# One-shot shortcut — describe + new combined (behaves like git commit)
jj commit -m "feat: add login"
```

Prefer **describe → new**: the two steps compose, and `jj commit` is sugar for the same pair.

> **Never** run `jj util snapshot` to "save" work. The working copy is always already snapshotted into `@`; `jj util snapshot` is a scripting helper that creates and names **no commit**. To finish work, use `jj describe` + `jj new`, or `jj commit`.

---

## Decision Tree

| Goal | Reference |
|------|-----------|
| All commands | `references/commands.md` |
| Revsets, filesets, templates | `references/revsets.md` |
| Git vs jj mapping | `references/git-differences.md` |
| Workflow patterns | `references/workflows.md` |
| Recovery procedures | `references/recovery.md` |

---

## LLM Rules

### ✅ Always
- `-m "message"` flag on all commands
- Change IDs (stable), not commit IDs
- One workspace per parallel task
- `jj git fetch` before push
- `jj duplicate` before destructive ops

### ❌ Never
- Interactive commands (`jj split -i`, `jj diffedit`)
- `git reset --hard` — use `jj undo`
- Share workspace paths between agents
- Skip `-m` flag (blocks on editor)
- `jj util snapshot` to "save" work — `@` auto-snapshots every command; finish work with `jj describe`/`jj new` or `jj commit`

### Multi-Agent Safety
- Workspace per agent: `jj workspace add --name <agent>-<task>`
- Coordinate via change IDs
- Never abandon shared commits
- Always fetch before rebase

---

## Quick Reference

**Symbols:** `@` (working copy), `@-` (parent), `@+` (child), `root()`, `trunk()`

**Revset ops:** `x::y` (range), `x..y` (descendants), `x|y` (union), `x&y` (intersect)

**Git→jj:** branch→bookmark, checkout→edit, add→(none, auto-track), stash→`jj new`

**Read `references/revsets.md` for full syntax →**
**Read `references/git-differences.md` for complete mapping →**

---

*Verify version with `jj version`. Refs target v0.41.0+. For workflows (staged commits, parallel agents, conflict resolution), see `references/workflows.md`*
