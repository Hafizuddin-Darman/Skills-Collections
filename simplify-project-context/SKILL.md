---
name: simplify-project-context
description: Simplify verbose project-context.md into lean CONTEXT.md files using a unified format. Use when user says "simplify context", "compress context", "align context format", "update CONTEXT.md", or "process project context".
---

# Simplify Project Context

Compress verbose source markdown file into lean new `CONTEXT.md` files.

## Quick Start

1. Read existing context source (ask user for source)
2. Load [TEMPLATE.md](TEMPLATE.md) for the target format
3. Extract facts from source into template sections
4. Write into new file (ask user for destination)

## Workflows

### A - Simplify Verbose Source (primary workflow)

1. Find source material
2. Extract all facts: stack versions, constraints, patterns, naming, edge cases, anti-patterns
3. Fill in [TEMPLATE.md](TEMPLATE.md) sections - skip sections with no content
4. Write to destination file
5. Make sure that every technical fact from the source appears in the output. No facts must be lost.

### B - Update Existing CONTEXT.md

1. Read current `CONTEXT.md`
2. Read TEMPLATE.md, compare section-by-section
3. Identify: missing sections, wrong formatting, misplaced content
4. Rewrite to match template - preserve ALL existing technical facts
5. Write updated file

### C - Align Existing to Unified Format

1. Read current `CONTEXT.md`
2. Read TEMPLATE.md, compare section-by-section
3. Identify: missing sections, wrong formatting, misplaced content
4. **Preserve ALL existing technical facts** — reorganize ONLY, never delete. Same rule as Workflow B.
5. Reorganize domain-scattered sections into pattern-grouped sections
6. Typical target: 120-200 lines, 5-8KB. You can exceed the cap when the file is fact-dense (many distinct patterns, reference tables, configuration examples). A 300-line CONTEXT.md that keeps every fact is better than a 150-line file that drops a host-call wrapper.

### Verification Pass (mandatory — runs after Workflows A, B, and C)

1. Enumerate which fact categories the source contains (see Fact Inventory under Non-negotiable)
2. For each category source has, make sure that the output has equivalent content
3. Any gap → re-add now, OR justify the drop (for example, "source perf rules were generic, not project-specific — dropped, flagged in CONTEXT.open-questions.md")
4. Report in chat: "Categories preserved: X/Y. Dropped with reason: Z."
5. If the output is less than 60 percent of the source line count, and the source has more than five distinct patterns, you probably dropped facts. Go back and re-add them.

## Rules

### Non-negotiable
- **Never lose facts.** Compression removes words, not information. When in doubt, keep — lean ≠ sparse.
- **No person/ticket references.** Use only references that are part of the specification (regulatory citation, PEN-test report, vendor document). Write rules directly: "use X, not Y". Do not write "senior says use X".
- **Uncertainty protocol.** If unsure whether a fact is project-specific implementation vs general knowledge, ask the user. Record unresolved items in `CONTEXT.open-questions.md` (same directory as `CONTEXT.md`).
- **Fact Inventory.** Every source item of these categories must appear in the output (own slot or merged into a related one):
  - **Mandated / banned API names** — prose + inline code span
  - **Version locks** — Stack table
  - **Reference catalogs** — table/list if source has one (controls, managers, services, banned lists, custom-DLL lists, file-layout tables)
  - **File-layout conventions** — table or short list
  - **Config / manifest formats** — inline key list (only required keys, not full file)
  - **Doc / comment standards** — table or list
  - **Naming conventions** — table
  - **Approach choosers** — keep when-to-use logic even when compressing code
  - **Host / integration gotchas** — anti-pattern row or Edge Case
  - **Build / deploy** — table or list
  - **Anti-pattern pairs** — every ❌/✅ row survives. Collect rows from ALL source sections.
- **Stack section = always a table** with Version column
- **Tagline = blockquote `>`** with status/warning on 2nd line
- **Anti-Patterns = always a table**, always the LAST section
- **Implementation Rules: name the API/approach. Use a code block only when prose + inline spans cannot carry the structure**

### Section structure
- **Canonical order:** Tagline → Stack → (Terminology) → Architecture → (Multi-Repo) → Implementation Rules → Security → Testing → (Checklist) → (Edge Cases) → (Naming Conventions) → Build & Deploy → (Directory Layout) → Anti-Patterns
- **Implementation Rules = one umbrella `## Implementation Rules`** with `### {Pattern}` subsections (DI, Entity, Page Lifecycle, WCF, Validation...). Group by pattern, NOT by domain.
  - *Polyglot exception:* a project with a hard language boundary can use top-level language groups instead — `## C# Patterns`, `## JavaScript Patterns` — each with `###` pattern subsections. Use only when the language split is the primary organizing axis.
  - *Many-subsystems exception:* a project integrating several distinct subsystems (each with its own conventions that do not overlap) can use one top-level section per subsystem (for example, `## UmbracoForms FieldType`, `## Session`, `## Database`). Use only when subsystem boundaries are more meaningful than pattern boundaries.
  - *Critical-domain exception:* a single domain rule that is critical to the project can have its own top-level section (for example, `## Custom Product Versioning (CRITICAL)`).
  - Default is the umbrella. Exceptions are opt-in. If in doubt, use the umbrella.
- **Security = top-level `## Security`** (not nested under Implementation Rules). It is a cross-cutting concern. Agents must find it fast.
- **Checklist heading = `## Checklist: {Thing}`** (for example, `## Checklist: New Page`). Do not write "New Page Checklist".
- **Naming Conventions = table** (Element | Convention | Example). Omit if standard with no project-specific quirks.

### Optional sections (include only when they have content)
- **Project Status** — maintenance/EOL/frozen callout. If it is one line, fold it into the tagline.
- **Multi-Repo** — mark `CRITICAL` when cross-repo changes can break siblings. Place right after Architecture.
- **Terminology** — domain glossary. Place right after Stack (define words before Architecture uses them).
- **Testing, Edge Cases, Directory Layout, File Header** — as needed.
