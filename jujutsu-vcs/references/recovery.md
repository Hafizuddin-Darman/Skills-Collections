# Jujutsu Recovery & Safety

Complete guide to recovery procedures, safety rules, and emergency procedures for Jujutsu (jj).

---

## The Safety Net: Operation Log

Every operation in jj is recorded in the **operation log**. This is your ultimate safety net.

### View Operation Log

```bash
jj op log                    # Full graph view
jj op log --no-graph        # Flat list (easier to read)
jj op log -T <template>     # Custom format
```

### Understand Operation Log

Each operation includes:
- **Operation ID**: Unique identifier (e.g., `qpqyxysx`)
- **Operation type**: What was done (new, rebase, describe, etc.)
- **Timestamp**: When it happened
- **Affected commits**: What changed

### Example Output

```
@  qpqyxysx  2024-01-15 10:30:45  new: 
|  |  
|  |  Commit ID: 1234567890ab
|  |  Description: feat: add login
|  
o  qprxxyzab  2024-01-15 10:25:00  describe: 
|  |  
|  |  Commit ID: abcdef1234
|  |  Description: initial
|  
...
```

---

## Recovery Ladder

Follow this ladder from simplest to most drastic:

### Level 1: Undo Last Operation

```bash
jj undo
```

**When:** You made a mistake in the last operation (wrong message, accidental edit)

**How:** Simply undo. Works for almost everything.

### Level 2: See What Happened

```bash
jj op log
```

**When:** You're not sure what went wrong

**How:** Browse the log to find the operation before things went wrong

### Level 3: Check Past State

```bash
# See what the repo looked like at an operation
jj --at-op <operation-id> status

# See what commits existed
jj --at-op <operation-id> log
```

**When:** You want to verify a past state before restoring

### Level 4: Restore to Operation

```bash
jj op restore <operation-id>
```

**⚠️ DANGER:** This restores to a past operation. It doesn't undo - it erases everything after.

**When:** You've made multiple mistakes and want to go back

**Important:**
- Always ask user confirmation first!
- Explain what will be lost
- Consider if `jj undo` is enough

---

## Common Recovery Scenarios

### Scenario 1: Accidental Message Change

```bash
# You did:
jj describe -m "wrong message"

# Fix:
jj describe -m "correct message"
```

Or if you realized immediately:
```bash
jj undo
jj describe -m "correct message"
```

### Scenario 2: Wrong Commit Edited

```bash
# You did:
jj edit <wrong-commit>

# Fix:
jj edit <correct-commit>
```

Or:
```bash
jj undo
jj edit <correct-commit>
```

### Scenario 3: Lost Commits After Rebase

```bash
# After failed rebase, commits seem gone

# Step 1: Find them in operation log
jj op log

# Step 2: Check if they exist
jj --at-op <pre-rebase-op> log

# Step 3: Restore if needed
jj op restore <pre-rebase-op>
```

### Scenario 4: Accidental Abandon

```bash
# You did:
jj abandon <important-commit>

# Find it:
jj op log
# Look for "abandon" operation

# Restore:
jj op restore <abandon-op>
# Or find the commit ID and edit:
jj edit <commit-id>
```

### Scenario 5: Workspace Issues

```bash
# Workspace shows stale
jj workspace update-stale

# Or remove and recreate
jj workspace forget <name>
jj workspace add <path>
```

### Scenario 6: Conflict Resolution Mess

```bash
# Made things worse during conflict resolution

# Step back
jj undo

# Try again with cleaner approach
jj resolve --list
# Fix one file at a time
jj resolve <file>
```

---

## Safety Rules

General agent rules (workspace per agent, change IDs, fetch before rebase, duplicate before destructive) live in `SKILL.md`. Below: recovery-specific approval gates.

### Forbid These Without Approval

These commands require explicit user permission:

```bash
# DANGEROUS - requires approval:
jj op restore <operation-id>    # Restores to past state
jj util gc                       # Garbage collection
jj bookmark delete <shared>     # Delete shared bookmark  
jj abandon <shared-commit>       # Abandon shared commit
```

### Emergency Git Bailout

In worst case (colocated repo):
```bash
# Keep .git, delete .jj
rm -rf .jj

# Use Git normally
git checkout .
git reset --hard
```

Then reinitialize jj:
```bash
jj git init --colocate
```

---

## Conflict Recovery

Made a mess during conflict resolution? The normal conflict-resolution steps live in `references/workflows.md` §7. Recovery angle only:

### Abandon Resolution Attempt

```bash
# Went wrong direction?
jj undo                    # Back to conflicted state
# retry with a different resolution
```

### Multiple Conflicts Overwhelming

Resolve one file at a time, `jj resolve --list` between each — don't batch.

---

## Pre-Operation Checklist

Before risky operations, verify:

```bash
# 1. Current state
jj st

# 2. Recent log
jj log -r @-:: --no-graph

# 3. Operation log (if unsure)
jj op log --no-graph
```

---

## Backing Up

### Export Working Copy State

```bash
# Show current state
jj show @ > /tmp/backup.txt
jj diff > /tmp/diff.patch
```

### Snapshot Before Risky Operation

```bash
# Duplicate current commit
jj duplicate @

# Now proceed with risky operation
# If wrong, abandon the result and edit the duplicate
```

---

## Emergency Commands Reference

| Problem | Solution |
|---------|----------|
| Last operation wrong | `jj undo` |
| Don't know what happened | `jj op log` |
| Check past state | `jj --at-op <id> status` |
| Go back in time | `jj op restore <id>` (ASK FIRST!) |
| Workspace stuck | `jj workspace update-stale` |
| Reset to known good | `jj duplicate <good-rev>` |
| Total mess | `rm -rf .jj` (colocated: keeps .git) |

---

## Getting Help

### Built-in Help

```bash
jj help
jj help <command>
jj help -k <keyword>
jj help -k revset
jj help -k bookmark
```

### Debug Commands

```bash
jj status --template 'change_id'
jj log -T 'change_id.short() ++ " " ++ description'
jj config list
```

---

## Best Practices

1. **Use `jj undo` first** - It's safe and instant
2. **Check `jj op log`** before panicking
3. **Duplicate before destructive** - Always `jj duplicate` before split/rebase
4. **Use change IDs** - They're stable
5. **Fetch before rebase** - `jj git fetch && jj rebase ...`
6. **Ask before dangerous ops** - Don't restore without permission

---

*For workflow patterns, see `references/workflows.md`. For command reference, see `references/commands.md`.*
