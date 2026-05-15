# Jujutsu Commands Reference

Complete reference for all Jujutsu (jj) commands. Organized by category for easy lookup.

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
jj st                         # Status (shorthand: jj s)
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
jj log --at <rev>             # Log at specific revision
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
jj cat -r <rev> <path>        # Show file at revision
jj file list -r <rev>         # List files in revision
jj file show -r <rev> <path>  # Show file at revision
jj file annotate <path>        # Blame (who changed each line)
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
jj new -A <author> <base>     # Create with specific author
```

### Describe (Commit Message)

```bash
jj describe -m "message"      # Set/update message (shorthand: jj desc)
jj describe -m "message" -r <rev>  # Describe specific revision
jj describe --reset           # Clear description
```

### Edit Existing

```bash
jj edit <rev>                 # Edit commit (switch working copy to it)
jj edit <rev> --no-edit       # Edit without switching
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
jj squash -into <target>      # Squash into specific commit
jj squash -r <rev>           # Squash revision into its parent
```

### Split

```bash
jj split -r <rev>             # Split revision by files (non-interactive)
jj split -r <rev> <paths>    # Split: paths go to first, rest to second
```

**Note:** Avoid interactive split (`jj split -i`). Use duplicate + split instead.

### Absorb

```bash
jj absorb                     # Auto-squash changes into closest ancestor commits
jj absorb --to <rev>          # Absorb into specific commit
```

### Rebase

```bash
jj rebase -s <src> -d <dest>      # Rebase source onto destination (with descendants)
jj rebase -s <src> -o <dest>      # New syntax (v0.36+): onto
jj rebase -r <rev> -d <dest>      # Rebase revision only (no descendants)
jj rebase -b <bookmark> -d <dest> # Rebase bookmark
jj rebase --skip-emptied          # Skip empty commits
jj rebase --preserve-timestamps   # Keep original timestamps
```

### Move

```bash
jj move <from> <to>           # Move file between commits
jj move <path> -d <dest>      # Move file to destination commit
```

---

## Bookmarks (Branches)

```bash
jj bookmark list              # List bookmarks (shorthand: jj b list)
jj bookmark create <name>    # Create bookmark
jj bookmark set <name>        # Move bookmark to working copy
jj bookmark set <name> -r <rev>  # Move bookmark to revision
jj bookmark delete <name>     # Delete bookmark
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
jj git push --force           # Allow moving bookmarks backwards
jj git push --dry-run         # Show what would be pushed
```

### Remote Management

```bash
jj remote list               # List remotes
jj remote add <name> <url>   # Add remote
jj remote remove <name>      # Remove remote
```

---

## Workspaces

```bash
jj workspace add <path>              # Create new workspace
jj workspace add --name <name> <path>  # With name
jj workspace add -r <rev> <path>    # At specific revision
jj workspace list                   # List workspaces
jj workspace forget <name>          # Remove workspace (keeps files)
jj workspace root                   # Show workspace root
jj workspace update-stale            # Update stale working copy
jj workspace delete <name>          # Delete workspace
```

---

## Discarding & Restoring

```bash
jj restore <paths>                  # Discard changes in working copy
jj restore --from <source> <paths>  # Restore files from another revision
jj restore -r <rev> <paths>        # Restore files from revision to working copy
jj restore -c <conflict> <paths>    # Restore conflict markers
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

**Important:** Always duplicate before destructive operations like split or rebase.

---

## Conflicts

```bash
jj resolve               # Resolve conflicts (interactive)
jj resolve <path>        # Resolve specific file
jj resolve --list       # List conflicted files
jj resolve --dry-run    # Check if resolvable
jj conflicts list       # List all conflicts
```

---

## Operations & Recovery

### Undo

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

### Restore from Operation

```bash
jj --at-op <operation-id> status   # Check state at operation
jj op restore <operation-id>       # Restore to operation (DANGEROUS!)
```

---

## Advanced Commands

### Fix

```bash
jj fix                  # Apply automatic fixes (formatting, etc.)
jj fix -r <rev>         # Fix specific revision
jj fix --fixes <tool>   # Run specific fixer
```

### Bisect

```bash
jj bisect start           # Start bisect
jj bisect bad             # Mark current as bad
jj bisect good <rev>      # Mark revision as good
jj bisect reset           # Reset bisect
```

### Sign

```bash
jj sign <rev>           # Cryptographically sign revision
jj verify <rev>         # Verify signature
```

### Tags

```bash
jj tag list             # List tags
jj tag create <name> -r <rev>  # Create tag
jj tag delete <name>    # Delete tag
```

### Util

```bash
jj util gc              # Garbage collect (rarely needed)
jj util exec <cmd>      # Execute external command
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
| `-v, --verbose` | Verbose output |
| `-q, --quiet` | Quiet output |
| `--color` | Color output (auto/never/always) |
| `--config` | Config override |

### Revision Selection

| Flag | Description |
|------|-------------|
| `-r`, `--revision` | Revision to operate on |
| `--revisions` | Revision range (note: NOT `-r` for ranges!) |
| `-s`, `--source` | Source revision for rebase |
| `-d`, `--destination` | Destination for rebase |
| `-o`, `--onto` | New parent for rebase (v0.36+) |

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

*This is a comprehensive reference. For workflow patterns and recovery procedures, see `references/workflows.md` and `references/recovery.md`.*
