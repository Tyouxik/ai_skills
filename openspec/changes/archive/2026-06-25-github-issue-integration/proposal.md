## Why

OpenSpec changes contain richly structured artifacts (proposal, design, tasks, specs) — but they exist only on the filesystem with no GitHub integration. Teams using GitHub Issues for project tracking have no automated bridge between OpenSpec's spec-driven workflow and GitHub's issue/PR lifecycle. Creating a project-agnostic `openspec-github` skill connects these two worlds: OpenSpec changes get mirrored as structured GitHub issues, implementation branches/PRs link back, and issue status stays synchronized throughout the change lifecycle. The skill operates at the user level (`~/.agents/skills/openspec-github/`) and works with any project using OpenSpec, regardless of language, framework, or domain.

## What Changes

- **New `openspec-github` OpenCode skill**: A companion skill that augments each existing OpenSpec stage (explore, propose, apply, archive) with GitHub operations via the `gh` CLI
- **Explore stage integration**: Pull GitHub issue content for exploration context (read-only, no creation)
- **Propose stage integration**: Create a structured GitHub issue from proposal/design/tasks artifacts OR link an existing issue to the change. Stores the `issue`→`change` relationship via `openspec set`.
- **Apply stage integration**: Always create a git branch and PR. Link the PR to the associated issue. Sync task progress to the issue checklist.
- **Archive stage integration**: Verify PR merge status, close the linked issue, then archive the change.
- **Labels convention**: OpenSpec capabilities are mapped to GitHub labels via the `spec:<capability-name>` format, applied automatically during propose and import stages.
- **Structured issue template**: A canonical format (`## Why`, `## What Changes`, `## Tasks`, `## Impact Areas`, `## Design Notes`) that is both human-readable and machine-parseable for bi-directional sync.
- **Link storage**: Change→Issue relationship stored via `openspec set github.issue <N>` and `openspec set github.repo <owner/repo>` in the change's `.openspec.yaml`.

## Capabilities

### New Capabilities
- `github-issue-export`: Create or update a structured GitHub issue from an OpenSpec change (proposal → issue title/body, tasks → checklist, capabilities → labels)
- `github-issue-import`: Pull a GitHub issue into OpenSpec — parse its body into proposal context, derive a change name, link the issue to the change
- `github-branch-pr`: Always create a git branch and draft PR during apply, linked to the associated issue
- `github-issue-track`: Bi-directional sync of task checklist progress between tasks.md and the linked GitHub issue body
- `github-archive-guard`: Verify PR merge status before closing the issue and archiving the change

### Modified Capabilities
<!-- No existing capabilities are modified — this is a new skill that augments the workflow without changing existing spec-level behavior. -->

## Impact

- **Skills**: New `openspec-github` skill installed at user level (`~/.agents/skills/openspec-github/SKILL.md`) with a `references/issue-template.md`. Developed and tested in this repo but packaged for global use across all projects.
- **CLI Dependencies**: Requires `gh` CLI (GitHub CLI) installed and authenticated. Uses `openspec set` for metadata storage.
- **Workflow**: Existing OpenSpec skills (explore, propose, apply, archive) remain unchanged. The `openspec-github` skill is invoked alongside them by the agent when GitHub integration is desired.
- **Metadata**: New fields in change `.openspec.yaml`: `github.issue` (issue number), `github.repo` (owner/repo)
- **No code dependencies**: Pure shell/skill workflow, no new packages or code files. Project-agnostic — works with any language, framework, or project structure.
- **Installation scope**: User-level skill (not project-specific). Reusable across all projects that use OpenSpec.
