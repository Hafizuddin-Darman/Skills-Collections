<!--
FACT PRESERVATION: Every mandated or banned API name, reference catalog, configuration
or manifest format, doc standard, naming convention, approach chooser, host gotcha, and
❌/✅ anti-pattern pair in the source MUST map to a slot below. If there is no slot,
create one or merge into the nearest pattern. See SKILL.md → Fact Inventory + Verification Pass.
-->

# {Name} — {One-Line Description}

> {Key context: frozen status, multi-repo, domain warning, or scope clarification. Omit if obvious.}

## Stack

| What | Version | Notes |
|------|---------|-------|
| ... | ... | **FROZEN** / critical notes |

## Terminology *(optional)*

{Domain glossary — terms the codebase uses that are not industry-standard. Omit if none.}

## Architecture

{Dependency graph, project layout, or multi-repo diagram. Use whatever best represents the project structure. If multi-repo is CRITICAL and large, break it out into the `## Multi-Repo` section below.}

## Multi-Repo *(optional — mark CRITICAL if cross-repo changes can break siblings)*

{Which repos feed this one, shared-assembly/DLL dependency flow, build-order caveats.}

## Implementation Rules

### {Pattern Name 1 — for example, DI, Entity, Page Lifecycle}

```{lang}
// ✅ Correct pattern
// ❌ Wrong pattern (if commonly mistaken)
```

### {Pattern Name 2}
...

{One umbrella section with `### {Pattern}` subsections (DI, Entity, Page Lifecycle, WCF...). Group by pattern, NOT by domain. Each subsection names the canonical API or approach. Include a code example only when prose + inline code spans cannot carry the structure. Do NOT create separate top-level sections for PDF, Session, Data Access, or other domains. Fold them into the relevant pattern group. Exceptions: polyglot (language groups), many-subsystems (one section per subsystem), critical-domain (standalone CRITICAL section). See SKILL.md for details.}

## Security

{Domain-specific security rules — access-control patterns, function-code gating, auth model. It is a cross-cutting concern. Keep it at the top level so it is found fast. Omit if none beyond standard practice.}

## Testing

{Framework, patterns, naming. Omit if project has no tests or only manual testing.}

## Checklist: {New Feature / New Page / New Operation}

1. Step one
2. Step two
...

{Only include if the project has clear, repeatable touch-points for new work.}

## Edge Cases

- **{Edge case name}:** description of gotcha / workaround
...

{Only include if project has non-obvious traps.}

## Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| ... | ... | ... |

{Omit this section if conventions are standard C#/VB.NET with no project-specific quirks.}

## Build & Deploy

{How to build, where outputs go, branch/commit conventions. Omit if trivial.}

## Directory Layout

```
{Project}/
├── {Folder}/     # {Purpose}
...
```

{Only include if the layout is non-obvious from the Architecture section.}

## Anti-Patterns

| ❌ | ✅ |
|----|----|
| ... | ... |

{Required. Always the LAST section. Even legacy projects have "do not upgrade X" entries.}
