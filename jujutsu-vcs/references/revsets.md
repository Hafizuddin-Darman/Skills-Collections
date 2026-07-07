# Jujutsu Revsets, Filesets & Templates

Complete reference for jj's three DSLs: Revsets (select revisions), Filesets (select files), and Templates (format output).

---

## Revsets: Revision Selection

Revsets select which revisions to operate on. They're used with `-r` flag.

### Basic Selectors

| Selector | Description |
|----------|-------------|
| `@` | Current working copy |
| `@-` | Parent of working copy |
| `@--` | Grandparent of working copy |
| `@+` | Child of working copy |
| `<change-id>` | Specific revision by change ID |
| `<commit-id>` | Specific revision by commit ID (avoid) |
| `root()` | The root commit (empty) |

### Ancestral Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `x-` | Parent of x | `@-` |
| `x--` | Grandparent of x | `@--` |
| `x+` | Child of x | `@+` |
| `x-N` | Nth parent of x | `@-3` |
| `x~N` | Nth parent of x | `main~5` |

### Range Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `x::y` | Ancestors of x through descendants of y (inclusive) | `main::@` |
| `x..y` | Descendants of x excluding y | `main..@` |
| `x:` | All descendants of x | `main:` |
| `::x` | All ancestors of x | `::main` |
| `x::` | All descendants of x | `main::` |

### Set Operations

| Operator | Description | Example |
|----------|-------------|---------|
| `x \| y` | Union: in either x or y | `main \| develop` |
| `x & y` | Intersection: in both x and y | `main & @` |
| `x ~ y` | Difference: in x but not in y | `@ ~ bookmark` |
| `~x` | Negation: not in x | `~mine()` |

---

## Revset Functions

### Ancestor Functions

```lua
parents(x)          -- Parents of x
children(x)         -- Children of x
ancestors(x)        -- All ancestors of x
descendants(x)      -- All descendants of x
ancestors(x, n)     -- N generations of ancestors
descendants(x, n)  -- N generations of descendants
roots(x)            -- Roots of x (no ancestors in x)
heads(x)            -- Heads of x (no children in x)
```

### Bookmark Functions

```lua
bookmarks()                    -- All local bookmarks
bookmarks(<pattern>)           -- Bookmarks matching pattern
remote_bookmarks()             -- All remote bookmarks
remote_bookmarks(<remote>)     -- Specific remote
tracked_remote_bookmarks()      -- Tracked remote bookmarks
```

### State Functions

```lua
empty()              -- Empty commits (no content changes)
conflicted()         -- Commits with conflicts
visible()            -- Visible commits (not hidden)
immutable()          -- Immutable commits (protected)
mutable()            -- Mutable commits (rewriteable)
```

### Metadata Functions

```lua
author(x)            -- Matches author (exact)
author.regex("pat")  -- Author matching regex
author.name("name")  -- Author name
author.email("email") -- Author email

committer(x)        -- Matches committer
committer.regex()
committer.name()
committer.date(after|x)

description(x)           -- Matches description (exact)
description.regex("pat") -- Description matching regex
description.substring("pat")  -- Contains substring

file(path)              -- Files in commit
file.regex("pat")       -- Files matching regex
diff_contains("pat")    -- Diff contains pattern
```

### Time Functions

```lua
committer_date(after|x)
committer_date(before|x)
committer_date(x)
author_date(after|x)
author_date(before|x)
```

### Special Sets

```lua
all()          -- All visible commits
none()         -- Empty set
trunk()        -- Main/trunk branch (default: main|master)
tags()         -- All tags
git_refs()     -- All Git references
```

### User Functions

```lua
mine()         -- Your commits (matches user.email)
mine().before(x)  -- Your commits before x
```

### Working Copy

```lua
working_copy()     -- Working copy commit
working_copy(<repo>)  -- Working copy in specific repo
```

---

## Common Revset Examples

### My Recent Work
```bash
jj log -r 'mine() & @-::'
jj log -r 'mine().before(@)'
```

### Not Yet Pushed
```bash
jj log -r 'remote_bookmarks()..'
jj log -r '@-:: - remote_bookmarks()::'
```

### Changes Since Main
```bash
jj log -r 'main..'
jj log -r 'main::'
```

### Parallel Branches
```bash
jj log -r 'children(main)'
jj log -r '@- & children(@--)'
```

### Find By Description
```bash
jj log -r 'description.regex("fix.*")'
jj log -r 'description.substring("auth")'
```

### Find By Author
```bash
jj log -r 'author("john@example.com")'
jj log -r 'author.regex("john.*")'
```

### Find By File
```bash
jj log -r 'file("src/main.rs")'
jj log -r 'file.regex("src.*\.rs$")'
```

### Empty Commits (TODOs)
```bash
jj log -r 'empty()'
jj log -r 'mine() & empty()'
```

### Commits with Conflicts
```bash
jj log -r 'conflicted()'
```

---

## Filesets: File Selection

Filesets select which files to operate on. Used with file-operating commands.

### Basic Fileset Syntax

```bash
# Glob patterns (default in v0.36+)
jj diff --fileset '*.rs'
jj diff --fileset 'src/**'
jj diff --fileset 'src/*.rs'

# Path literals
jj diff --fileset 'src/main.rs'

# Combine with operators
jj diff --fileset '*.rs & src/**'
```

### Fileset Functions

```lua
glob("pattern")       -- Glob pattern
path("literal")       # Literal path
regex("pattern")      # Regex match
```

### Examples
```bash
# All Rust files
jj diff --fileset 'glob("*.rs")'

# In src directory
jj diff --fileset 'glob("src/**")'

# Specific file
jj diff --fileset 'path("src/main.rs")'
```

---

## Templates: Output Formatting

Templates control output format for commands like `jj log`.

### Template Language

```lua
-- Simple fields
change_id
change_id.short()
description
description.first_line()
description.first_line().chars(50)

-- Author/committer
author.name()
author.email()
author.date()
committer.name()
committer.date()

-- Files
file_paths
file_paths.len()

-- Graph
change_id
" " 
description.first_line()
```

### Template Functions

```lua
-- String
"text".upper()
"text".lower()
"text".len()
"text".chars(n)
"text".substring("pat")
"text".regex("pat")
"text".contains("pat")
"text".strip_last_line()

-- Conditional
if(condition, "yes", "no")
if(condition, "yes")

-- Collection
join("sep", collection)
len(collection)
```

### Graph Templates

```bash
# Default
jj log -T 'change_id.short() ++ " " ++ description.first_line()'

# With author
jj log -T 'author.name() ++ " | " ++ description.first_line()'

# Compact
jj log -T 'change_id.short() ++ "|" ++ description.first_line() ++ "|" ++ file_paths.len()'

# With files
jj log -T 'change_id.short() ++ " " ++ description.first_line() ++ "\n" ++ file_paths'
```

### Complex Example

```bash
jj log -T '
concatenate(
  change_id.short(),
  " | ",
  if(description, description.first_line(), "(empty)"),
  " | ",
  author.name(),
  " | ",
  committer.date().format("%Y-%m-%d")
)'
```

---

## Common Command Combinations

### Show Only My Recent Changes
```bash
jj log -r 'mine() & @-::' -p
```

### Find Unpushed Changes
```bash
jj log -r 'remote_bookmarks()..' --reversed
```

### See What Changed in a File
```bash
jj log -r 'file("src/main.rs")' -p
```

### List All Branches with Unpushed
```bash
jj bookmark list
jj log -r 'remote_bookmarks()..' --reversed -T 'bookmark_names() ++ " " ++ description.first_line()'
```

### Show commits touching specific path
```bash
jj log -r 'path("src/utils.rs")'
```

### Check for empty commits
```bash
jj log -r 'empty() & mine()'
```

---

## Common Mistakes

1. **`-r` vs `--revisions`**: Use `-r` for single revision, `--revisions` (NOT `-r`) for range
   - ✅ `jj log -r main..`
   - ❌ `jj log --revisions main..`

2. **Using commit IDs**: Use change IDs (stable) not commit IDs (volatile)
   - ✅ `jj edit qzvzulzz`
   - ❌ `jj edit abc123def456`

3. **Pipe in shell vs revset**: The `|` is a shell pipe unless the revset is quoted
   - ✅ `jj log -r 'main | develop'` (quoted — revset union)
   - ❌ `jj log -r main | develop` (unquoted — shell parses `|` as pipe)

4. **Glob in fileset**: In v0.36+, globs are default. Explicit: `glob("*.rs")`

5. **Function vs field**: Some have both forms
   - `author.name()` (function) vs `author().name()` (field method)
   - Use what works; jj is flexible

---

## Quick Reference Card

| Task | Revset |
|------|--------|
| Working copy | `@` |
| Parent | `@-` |
| Grandparent | `@--` |
| All ancestors | `::@` |
| All descendants | `@::` |
| My commits | `mine()` |
| Not pushed | `remote_bookmarks()..` |
| Empty (TODOs) | `empty()` |
| With conflicts | `conflicted()` |
| Main branch | `trunk()` |
| All | `all()` |

---

*For comprehensive Git-to-jj command mapping, see `references/git-differences.md`. For workflow patterns, see `references/workflows.md`.*
