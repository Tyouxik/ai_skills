## Context

OpenSpec provides a spec-driven development workflow: `explore` → `propose` → `apply` → `archive`. Each stage produces structured artifacts (proposal.md, design.md, tasks.md, specs/). However, these artifacts live only on the filesystem — there is no GitHub integration. Teams that use GitHub Issues for project tracking lack an automated bridge between the spec-driven workflow and GitHub's issue/PR lifecycle.

**Current state:**
- OpenSpec skills (`openspec-explore`, `openspec-propose`, `openspec-apply-change`, `openspec-archive-change`, `openspec-sync-specs`) operate entirely within the local filesystem
- No `gh` CLI usage in any existing skill
- Change metadata is stored in `.openspec.yaml` per change directory
- The project is a solo Flutter app repo using `openspec` CLI

**Constraints:**
- Must work with existing `gh` CLI (no new dependencies)
- Must integrate with existing OpenSpec skills without modifying them
- Skill must be project-agnostic — works with any project using OpenSpec, regardless of language, framework, or domain
- Skill installed at user level (`~/.agents/skills/openspec-github/SKILL.md`) for cross-project reuse
- Link storage must use `openspec set` for metadata

## Goals / Non-Goals

**Goals:**
- Create a single `openspec-github` skill that augments all four OpenSpec stages with GitHub operations
- Provide structured, parseable GitHub issue format (bi-directional sync)
- Always create git branches and PRs during the apply stage
- Store change↔issue links via `openspec set github.issue <N>` and `openspec set github.repo <owner/repo>`
- Map OpenSpec capabilities to GitHub labels using `spec:<capability-name>` convention
- Verify PR merge status before closing issues and archiving changes

**Non-Goals:**
- Modifying existing OpenSpec skills (they remain unchanged; this skill is a companion)
- Project-specific behavior (the skill MUST work identically across all projects)
- Supporting GitHub Projects (initial scope is Issues + PRs only)
- Supporting multiple repos per change (one change → one repo)
- Cursor-specific skill format (OpenCode only; Cursor is a future consideration)
- Auto-commenting on issues for every task completion (only on major milestones)

## Decisions

### Decision 1: Single skill, not multiple

**Chosen:** One `openspec-github` skill with four stage sections (explore, propose, apply, archive).

**Alternatives considered:**
- **Three separate skills** (import/export/track): Abandoned because GitHub integration is not a parallel pipeline — it augments each stage sequentially. A single skill keeps all GitHub logic co-located and avoids triggering ambiguity.
- **Modifying existing skills**: Rejected because the existing skills are maintained independently. A companion skill approach avoids forks.

### Decision 2: `openspec set` for link storage

**Chosen:** Store issue number and repo in the change's `.openspec.yaml` via `openspec set github.issue <N>`.

**Alternatives considered:**
- **HTML comment in issue body**: Unreliable — users can edit issue bodies. Used as a secondary signal, not primary storage.
- **Separate `.github` file**: Adds file proliferation. `openspec set` integrates with existing CLI.
- **Project-level JSON mapping**: Creates merge conflicts in team settings.

### Decision 3: Always create branch + PR on apply

**Chosen:** Non-negotiable — every apply creates a branch named `<change-name>` and a PR linked to the issue. No prompt, no skip option.

**Rationale:** Consistency. Every change is traceable via PR. Small changes that "don't need a PR" still benefit from CI validation, review history, and issue auto-linking.

### Decision 4: Structured issue format

**Chosen:** Issues use a canonical markdown format:

```markdown
## Why
[context + motivation]

## What Changes
[bulleted list of changes]

## Tasks
- [ ] Task description

## Impact Areas
[affected systems/code]

## Design Notes
[key architectural decisions]
```

**Rationale:** This format is both human-readable (renders well on GitHub) and machine-parseable (each `##` section maps to an OpenSpec artifact). The `openspec-issue-import` stage can reliably extract sections back into proposal.md and tasks.md.

### Decision 5: Labels convention

**Chosen:** OpenSpec capabilities map to `spec:<capability-name>` labels. Applied during propose (export) and read during explore (import).

**Rationale:** Prefix avoids namespace collisions with other label systems. The mapping is deterministic and reversible.

### Decision 6: Skill structure and installation

**Chosen:** The skill's SKILL.md is organized by OpenSpec stage, with each stage describing what GitHub operations to perform. The skill is developed in this repo for testing but installed at the user level for cross-project use.

```
~/.agents/skills/openspec-github/    ← Installed here (user-level, project-agnostic)
├── SKILL.md
└── references/
    └── issue-template.md
```

**Rationale:** User-level installation ensures the skill is available to every project. It uses only global tools (`gh`, `openspec`, `git`) — no project-specific paths, languages, or frameworks. The skill queries the current working directory's git remote and openspec configuration, adapting to whichever project invokes it.

## Risks / Trade-offs

| Risk | Mitigation |
|---|---|
| `gh` CLI not installed or not authenticated | Skill checks `gh auth status` at start of each stage; exits with clear message if unavailable |
| Issue body edited manually, breaking parseability | Import stage checks for expected section headers; falls back to treating entire body as "Why" if sections missing |
| PR not merged when archive runs | Archive stage checks `gh pr view --json state,mergedAt`; warns if unmerged, asks user whether to proceed |
| Change name collision with existing branch | Apply stage checks `git branch --list <name>` before creating; appends `-2` if conflict |
| `openspec set` not available (older CLI) | Skill detects CLI capabilities and falls back to writing a `.github` metadata file in the change directory |
| Multiple issues linked to one change | Only one issue per change is supported. If detect multiple links, skill uses `github.issue` stored in `.openspec.yaml` as source of truth |
