# Jujutsu Commands Reference

Complete reference for all Jujutsu (jj) commands (v0.41.0). Organized by category for easy lookup.

---

## Repository Setup

### Initialization

```bash
jj git init                    # Initialize jj repo (creates .jj)
jj git init --colocate        # Initialize in existing Git repo (.jj + .git)
jj git init --git              # Initialize with jj storing in .git (experimental)
```

### Configuration

```bash
jj config list                 # List all config
jj config get <key>           # Get config value
jj config set <key> <value>   # Set config (user-level)
jj config set --global <key> <value>  # Set global config
jj config unset <key>         # Unset a config option
jj config edit                # Open config in editor
jj config path                # Show config file path
```

Common config:
```toml
# ~/.jjconfig.toml
user.name = "Your Name"
user.email = "you@example.com"
```

---

## Viewing State

### Status

```bash
jj st                         # Status (shorthand)
jj status                     # Full status
jj status --template <tmpl>  # Custom template
```

### Log

```bash
jj log                        # Show log (default: @::)
jj log -r <revset>            # Show specific revisions
jj log --no-graph            # Flat list
jj log -p                     # Show patches
jj log --reversed             # Oldest first
jj log -T <template>         # Custom template
```

Template examples:
```bash
jj log -T 'change_id.short() ++ " " ++ description.first_line()'
jj log -T 'author.name() ++ " | " ++ description.first_line()'
```

### Diff

```bash
jj diff                       # Diff working copy vs parent
jj diff --git                 # Git-compatible diff
jj diff -r <rev>              # Diff revision vs its parent
jj diff -r <a>..<b>           # Diff between revisions
jj diff --stat                # Summary only
jj diff <path>                # Diff specific file
```

### Showing Content

```bash
jj show <rev>                 # Show commit details + diff
jj show <rev> --template <tmpl>  # Custom template
jj file list -r <rev>         # List files in revision
jj file show -r <rev> <path>  # Show file at revision
jj file annotate <path>       # Blame (who changed each line)
jj file search -r <rev> <pattern>  # Search content in files
```

---

## Creating Commits

### New Commit

```bash
jj new                        # Create new empty commit
jj new -m "message"           # Create with description
jj new <base-rev>             # Create child of specific revision
jj new <base-rev> -m "msg"    # Create from base with message
jj new --no-edit <base>       # Create but stay on current
jj new -A <after-rev>         # Insert new commit after given revision(s)
jj new -B <before-rev>        # Insert new commit before given revision(s)
jj new <a> <b>                # Create merge commit with two parents
```

### Describe (Commit Message)

```bash
jj describe -m "message"      # Set/update message (shorthand: jj desc)
jj describe -m "message" -r <rev>  # Describe specific revision
jj describe --stdin           # Read description from stdin
jj describe --editor          # Force open editor
```

### Metaedit

```bash
jj metaedit                   # Edit metadata without changing content
jj metaedit -m "message"      # Update message only
```

### Edit Existing

```bash
jj edit <rev>                 # Edit commit (switch working copy to it)
```

### Commit (Combined)

```bash
jj commit -m "message"        # Describe current + create new working copy
```

---

## Modifying Commits

### Squash

```bash
jj squash                     # Squash all changes into parent
jj squash -m "message"        # Squash with new message
jj squash --into <target>     # Squash into specific commit (alias: --to, -t)
jj squash -r <rev>            # Squash revision into its parent
jj squash --from <rev>        # Squash from specific revision
jj squash -k                  # Keep source revision (don't abandon)
```

Experimental squash (creates new commit):
```bash
jj squash -o <parent>         # Create new commit at parent
jj squash -A <after>          # Insert new commit after
jj squash -B <before>         # Insert new commit before
```

### Split

```bash
jj split -r <rev>             # Split revision (opens diff editor)
jj split -r <rev> <paths>    # Split: paths go to first, rest to second
jj split -r <rev> --parallel  # Split into sibling commits instead of parent/child
```

**Note:** Avoid interactive split (`jj split -i`). Use duplicate + split instead.

### Absorb

```bash
jj absorb                     # Auto-squash changes into closest ancestor commits
jj absorb --to <rev>          # Absorb into specific commit
```

### Rebase

```bash
jj rebase -s <src> -o <dest>      # Rebase source onto destination (with descendants)
jj rebase -r <rev> -o <dest>      # Rebase revision only (no descendants)
jj rebase -b <branch> -o <dest>   # Rebase branch
jj rebase --skip-emptied          # Skip empty commits after rebase
jj rebase -A <target>             # Insert after target (reorder)
jj rebase -B <target>             # Insert before target (reorder)
jj rebase --simplify-parents      # Simplify parent edges during rebase
```

Note: `-d` is an alias for `-o` (`--onto`). `-o` is the preferred flag.

### Parallelize

```bash
jj parallelize <revs>         # Make revisions siblings instead of parent/child
```

### Arrange

```bash
jj arrange                    # Interactively arrange the commit graph
```

---

## Bookmarks (Branches)

```bash
jj bookmark list              # List bookmarks (shorthand: jj b list)
jj bookmark create <name>    # Create bookmark
jj bookmark set <name>        # Create or move bookmark to working copy
jj bookmark set <name> -r <rev>  # Move bookmark to revision
jj bookmark move <name>       # Move bookmark to target revision
jj bookmark move <name> -r <rev> # Move bookmark to specific revision
jj bookmark advance           # Advance closest bookmark to target
jj bookmark delete <name>     # Delete bookmark (propagates to remote on push)
jj bookmark forget <name>     # Forget bookmark without deletion push
jj bookmark track <name>@<remote>  # Track remote bookmark
jj bookmark untrack <name>@<remote>  # Untrack
jj bookmark rename <old> <new>  # Rename bookmark
```

---

## Remote Operations

### Fetch

```bash
jj git fetch                  # Fetch from default remote
jj git fetch --remote <name>  # Fetch from specific remote
jj git fetch --all            # Fetch all remotes
```

### Push

```bash
jj git push                   # Push all bookmarks
jj git push --bookmark <name> # Push specific bookmark
jj git push --remote <name>   # Push to specific remote
jj git push --dry-run         # Show what would be pushed
```

### Remote Management

```bash
jj git remote list               # List remotes
jj git remote add <name> <url>   # Add remote
jj git remote remove <name>      # Remove remote
jj git remote rename <old> <new> # Rename remote
jj git remote set-url <name> <url>  # Set remote URL
```

### Colocation

```bash
jj git colocation status     # Check colocation status
jj git colocation enable     # Enable colocation
jj git colocation disable    # Disable colocation
```

---

## Workspaces

```bash
jj workspace add <path>              # Create new workspace
jj workspace add --name <name> <path>  # With name
jj workspace add -r <rev> <path>    # At specific revision
jj workspace list                   # List workspaces
jj workspace forget <name>          # Remove workspace (keeps files)
jj workspace rename <new-name>      # Rename current workspace
jj workspace root                   # Show workspace root
jj workspace update-stale            # Update stale working copy
```

---

## Navigating

```bash
jj next                       # Move working copy to child revision
jj next <n>                   # Move n children forward
jj prev                       # Move working copy to parent revision
jj prev <n>                   # Move n parents back
```

---

## Discarding & Restoring

### Restore

```bash
jj restore <paths>                  # Discard changes in working copy
jj restore --from <source> <paths>  # Restore files from another revision
jj restore -r <rev> <paths>        # Restore files from revision to working copy
```

### Revert

```bash
jj revert <rev>               # Apply the reverse of a revision's changes
jj revert <rev1> <rev2>       # Revert multiple revisions
```

### Abandon

```bash
jj abandon <rev>            # Abandon revision (keeps children)
jj abandon <rev>..         # Abandon revision and descendants
jj abandon @               # Abandon working copy changes only
```

---

## Duplicate

```bash
jj duplicate <rev>                # Duplicate revision (creates copy)
jj duplicate <rev>..<rev>         # Duplicate range
```

---

## Conflicts

```bash
jj resolve               # Resolve conflicts (interactive)
jj resolve <path>        # Resolve specific file
jj resolve --list       # List conflicted files
```

---

## Operations & Recovery

### Undo/Redo

```bash
jj undo                 # Undo last operation
jj redo                 # Redo undone operation
```

### Operation Log

```bash
jj op log               # View operation history
jj op log --no-graph    # Flat list
jj op log -T <template> # Custom template
jj op show <id>         # Show operation details
```

### Operation Management

```bash
jj --at-op <operation-id> status   # Check state at operation
jj op restore <operation-id>       # Restore to operation (DANGEROUS!)
jj op abandon <operation-id>       # Abandon operation history
jj op diff <op1> <op2>             # Compare changes between operations
jj op integrate <operation-id>     # Make operation part of the log
jj op revert <operation-id>        # Create new operation reverting an earlier one
```

---

## Evolog

```bash
jj evolog                    # Show how a change has evolved over time (alias: evolution-log)
jj evolog -T <template>      # Custom format
```

---

## Advanced Commands

### Fix

```bash
jj fix                  # Apply automatic fixes (formatting, etc.)
jj fix -s <rev>         # Fix specified revision(s) and descendants
jj fix --all-lines      # Format all lines instead of only modified
```

### Bisect

```bash
jj bisect run <command>    # Run command to find first bad revision
jj bisect run --range <revset> <command>  # Bisect within range
```

Note: Exit codes determine good/bad: 0 = good, non-zero = bad, 125 = skip.

### Sign/Unsign

```bash
jj sign                      # Cryptographically sign revision (uses revsets.sign)
jj sign -r <rev>             # Sign specific revision
jj unsign                    # Drop cryptographic signature
jj unsign -r <rev>           # Unsign specific revision
```

### Tags

```bash
jj tag list             # List tags
jj tag set <name> -r <rev>  # Create or update tag
jj tag delete <name>    # Delete tag
```

### Simplify Parents

```bash
jj simplify-parents <rev>    # Remove redundant parent edges
jj simplify-parents -r <revset>  # Simplify multiple revisions
```

### Sparse

```bash
jj sparse list          # List sparse patterns
jj sparse set <patterns>  # Set sparse patterns
jj sparse edit          # Edit sparse patterns in editor
jj sparse reset         # Reset to full checkout
```

### File Operations

```bash
jj file chmod +x <path>       # Set executable bit
jj file chmod -x <path>       # Remove executable bit
jj file track <path>          # Start tracking path
jj file untrack <path>        # Stop tracking path
```

### Gerrit

```bash
jj gerrit upload            # Upload to Gerrit Code Review
```

### Util

```bash
jj util gc              # Garbage collect (rarely needed)
jj util exec <cmd>      # Execute external command
jj util completion <shell>  # Generate shell completions
jj util config-schema   # Print JSON schema for config
jj util markdown-help   # Print CLI help in Markdown
jj util snapshot        # Snapshot working copy if needed
jj version              # Show version
jj help                 # Show help
jj help <command>       # Help for specific command
jj help -k <keyword>    # Search help
```

---

## Flags Reference

### Common Global Flags

| Flag | Description |
|------|-------------|
| `-r, --revision` | Revision(s) to operate on |
| `-r <revset>` | Revision set expression |
| `-T, --template` | Template for output |
| `--no-pager` | Disable pager |
| `--quiet` | Quiet output |
| `--color` | Color output (auto/never/always/debug) |
| `--config <NAME=VALUE>` | Config override |
| `--config-file <PATH>` | Additional config file |
| `--at-operation <ID>` | Load repo at operation (alias: --at-op) |
| `--ignore-working-copy` | Don't snapshot/update working copy |
| `--ignore-immutable` | Allow rewriting immutable commits |

### Revision Selection

| Flag | Description |
|------|-------------|
| `-r`, `--revision` | Revision to operate on |
| `-s`, `--source` | Source revision for rebase |
| `-o`, `--onto` | Destination for rebase (alias: -d) |
| `-b`, `--branch` | Branch to rebase |
| `-A`, `--insert-after` | Insert after target |
| `-B`, `--insert-before` | Insert before target |

---

## Environment Variables

| Variable | Description |
|----------|-------------|
| `JJ_USER` | Override user name |
| `JJ_EMAIL` | Override user email |
| `JJ_ROOT` | Override repo root |
| `EDITOR` | Editor for interactive commands |
| `VISUAL` | Editor (fallback) |
| `JJ_NO_PROGRESS` | Disable progress indicators |

---

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Generic error |
| 128 | No repository found |
| 255 | Internal error |

---

*This is a comprehensive reference for jj v0.41.0. For workflow patterns and recovery procedures, see `references/workflows.md` and `references/recovery.md`.*
