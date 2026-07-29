# Examples from Existing CONTEXT.md Files

These are annotated excerpts showing how each section looks in practice. Use as reference when filling in the template.

---

## Minimal Example: LeBlender (Small, Maintenance-Only Project)

```markdown
# LeBlender — Umbraco Grid Editor Extension

> Archived GitHub repo, forked & customized for Company Maintenance mode only. .NET 4.6.2, AngularJS 1.x.

## Project Status: MAINTENANCE MODE

Part of Company CMS ecosystem. See `programs/Company_CMS/CONTEXT.md` for shared rules.
Original GitHub repo is archived — changes in this Company fork only.

## Stack

| What            | Version | Notes                 |
| --------------- | ------- | --------------------- |
| .NET Framework  | 4.6.2   | **FROZEN**            |
| Umbraco CMS     | 7.15.7  | **FROZEN**            |
| AngularJS       | 1.x     | Built-in to Umbraco 7 |

## Anti-Patterns

| ❌                          | ✅                        |
| --------------------------- | ------------------------- |
| Modify archived GitHub repo | This Company fork only    |
| .NET Core patterns          | .NET 4.6.2                |
```

Note: Small project = fewer sections. Stack + Anti-Patterns is the minimum viable CONTEXT.md.

---

## Tagline Examples

| Project | Tagline |
|---------|---------|
| FrontEnd | `> Internal staff portal for core banking operations. VB.NET, .NET 4.8, WebForms.` |
| IntSvc | `> Middleware between frontend apps and Company core. WCF SOAP + Web API. Mixed VB.NET/C#.` |
| LeBlender | `> Archived GitHub repo, forked & customized for Company. Maintenance mode only.` |
| MVP3 | `> My Viewpoint 3. Customer-facing online banking web app. VB.NET, .NET 4.8, WebForms.` |
| ProdMgr | `> **NOT standard Merchello.** Product versioning completely replaced with custom implementation.` |
| CMS.Ext | `> C# backend for Company_CMS. Controllers, services, field types, workflows.` |
| CMS | `> Umbraco 7.15.7 public-facing website. Frontend delivery repo in a 3-repo architecture.` |

Key patterns:
- Always include primary language + framework in tagline
- Use **bold** for critical warnings (custom fork, multi-repo, not-what-you-think)
- State the role if not obvious from project name

---

## Stack Table Examples

### Standard (most projects)

```markdown
## Stack

| What               | Version | Notes                                                |
| ------------------ | ------- | ---------------------------------------------------- |
| VB.NET             | 9.0+    | ~85% of codebase. C# for tests only                  |
| .NET Framework     | 4.8     | **FROZEN**                                           |
| Enterprise Library | 4.1     | Logging, caching, validation                         |
```

Rules:
- Version column always present (use "GAC", "pre-5", "current" if exact unknown)
- **FROZEN** in Notes = never suggest upgrades
- Include custom/legacy libs with their critical constraints
- Put primary language/runtime first

### Multi-Repo (CMS Extension)

```markdown
## Stack

| What                        | Version                                          | Notes                                                     |
| --------------------------- | ------------------------------------------------ | --------------------------------------------------------- |
| .NET Framework              | 4.6.2 (most) / 4.7.2 (Security, Forms.Extension) | **FROZEN**                                                |
| AutoMapper                  | 3.3.1                                            | **STATIC API** — `Mapper.CreateMap<>()`, `Mapper.Map<>()` |
```

Rules:
- Multiple target frameworks in one row if needed
- Bold the API style if it is a common mistake point

---

## Architecture Examples

### Dependency Graph (LoanKeyFacts)

```markdown
## Architecture

UDA.LoanKeyFacts.Web
  ├── UDA.LoanKeyFacts.Services (project ref)
  └── UDA.LoanKeyFacts (project ref)
       └── UDA.LoanKeyFacts.Services (project ref)

| Project | Purpose |
|---------|---------|
| `.Services` | Interfaces + Entity DTOs |
| `.LoanKeyFacts` | Calculator + PDF report generation |
```

### Layer Table (IntegrationServices)

```markdown
## Architecture (6 Layers)

| #   | Project                   | Lang | Purpose                                   |
| --- | ------------------------- | ---- | ----------------------------------------- |
| 1   | Uda.DataAccess            | VB   | Data access, host services, core entities |
| 2   | Tpa.Services.*            | VB   | WCF service contracts & implementations   |
```

### Multi-Repo (Company_CMS)

```markdown
## Multi-Repo Architecture (CRITICAL)

| Repo                     | What Goes Here                                             |
| ------------------------ | ---------------------------------------------------------- |
| **Company_CMS** (this)   | `.cshtml`, `.js`, `.css`, `.html`, `.scss` — frontend only |
| **CompanyCMS.Extension** | `.cs` — controllers, services, field types, workflows      |

DLLs must go to **BOTH** `Dependencies/` and `bin/` — missing either = deployment incomplete.
```

Rules:
- Pick the representation that fits: graph, table, or diagram
- Multi-repo = always show the dependency flow with bold warning
- Include language per project if mixed (VB/C#)

---

## Implementation Rules Examples

### Pattern-Grouped (FrontEnd)

```markdown
### DI (Mandatory Pattern)
```vb
' ✅ Service Locator
_daDormancy = UDAUnity.Resolve(Of IDormancy)()

' ❌ NOT constructor injection, NOT Microsoft.Extensions.DependencyInjection
```

### Entity Pattern (Business Entities)
```vb
Public Class ChargeFeesRequest
    Inherits Data.Service  ' MUST inherit for host communication
    ...
End Class
```

### Security Patterns
```vb
' Approach 1: Function code check
If UDAWeb.Utils.CheckFunction("GCS252") Then BuildUI()
```
```

Rules:
- Each sub-heading = one pattern (DI, Entity, Validation, Security, and more)
- ✅/❌ inline comments for common mistakes
- Code block always inside the pattern section, never in a separate top-level section
- Domain-specific details (PDF, Session, XML) fold INTO the relevant pattern, not as standalone sections

### Domain-Specific (Product_Manager versioning)

```markdown
### Custom Product Versioning (CRITICAL)

Standard Merchello docs do NOT apply to versioning.

- **GUID-based `VersionKey`** — not incremental integers
- **Insert-only** — never modify existing version records
- **Master variant** per version (`Master = true`)

```csharp
// ❌ NEVER modify existing version records
Database.Update(productVersionDto);

// ✅ Always insert new records
Database.Insert(newVersionDto);
```
```

---

## Anti-Patterns Examples

### Simple (LeBlender)

```markdown
## Anti-Patterns

| ❌                          | ✅                        |
| --------------------------- | ------------------------- |
| Modify archived GitHub repo | This Company fork only    |
| .NET Core patterns          | .NET 4.6.2                |
| BEM CSS naming              | kebab-case                |
```

### Detailed (Product_Manager)

```markdown
## Anti-Patterns

| ❌                                     | ✅                                 |
| -------------------------------------- | ---------------------------------- |
| Upgrade AutoMapper                     | Keep 3.3.1 — static API everywhere |
| Standard Merchello docs for versioning | This is custom                     |
| Modify existing version records        | Insert-only                        |
| Incremental version numbers            | GUID `VersionKey`                  |
| Physical delete                        | Soft delete (`IsDeleted`)          |
```

Rules:
- Always present
- ❌ = what not to do (specific, not vague)
- ✅ = what to do instead (specific enough to act on)
- Cover version locks, API style mismatches, ORM prohibitions, naming violations
