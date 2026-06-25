# Structured Issue Template

This is the canonical format for OpenSpec-generated GitHub issues. Use this template when creating an issue from an OpenSpec change (Propose Export stage). The format is designed to be both human-readable on GitHub and machine-parseable for the Import stage.

```markdown
## Why
<!-- Context and motivation for this change. What problem does it solve? Why now? -->

## What Changes
<!-- Specific changes being made. Bullet list of new capabilities, modifications, or removals. -->
<!-- Mark breaking changes with **BREAKING**. -->

## Tasks
<!-- Implementation checklist. Mirror of tasks.md checkboxes. -->
- [ ] Task group 1: Description
  - [ ] Subtask 1.1: Detail
  - [ ] Subtask 1.2: Detail
- [ ] Task group 2: Description

## Impact Areas
<!-- Affected code, APIs, dependencies, systems. Bullet list. -->

## Design Notes
<!-- Key architectural decisions, tradeoffs, constraints. From design.md Decisions section. -->

---
<!-- openspec:change=<change-name> -->
<!-- repo:<owner/repo> -->
*Generated from OpenSpec change `<change-name>`*
```

## Section Mapping

| Issue Section | OpenSpec Artifact | Direction |
|---|---|---|
| `## Why` | `proposal.md` → Why | Export: copied as-is |
| `## What Changes` | `proposal.md` → What Changes | Export: copied as-is |
| `## Tasks` | `tasks.md` checkboxes | Bidirectional: synced on every update |
| `## Impact Areas` | `proposal.md` → Impact | Export: copied as-is |
| `## Design Notes` | `design.md` → Decisions | Export: summarized |
| Labels: `spec:<name>` | `proposal.md` → Capabilities | Bidirectional |

## Parsing Rules (Import Stage)

When reading an issue for import:
1. Detect sections by `## ` headers
2. Section content extends until the next `## ` header or end of body
3. HTML comments (`<!-- ... -->`) are metadata, not content
4. If no structured sections found, treat entire body as `## Why`
5. Checklist items are lines starting with `- [ ]` or `- [x]` or `* [ ]`
